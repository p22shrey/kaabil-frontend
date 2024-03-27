<script>
	import Button from "@smui/button";
	import Textfield from "@smui/textfield";
	import Card from "@smui/card";

	let videoUrl = "";
	let videoId = "";
	let player = undefined;
	let questions = [];
	let question = "";
	let currentQuestionIndex = 0;
	let transcript = "";
	let userResponse = "";

	function loadVideo() {
		videoId = extractVideoId(videoUrl);
		if (videoId) {
			if (player === undefined) {
				player = new YT.Player("player", {
					height: "360",
					width: "640",
					videoId: videoId,
					events: {
						onReady: onPlayerReady,
						onStateChange: onPlayerStateChange,
					},
				});
			} else {
				player.cueVideoById(videoId);
			}
		}
	}

	function extractVideoId(url) {
		const regex = /[?&]v=([^&#]+)/;
		const match = url.match(regex);
		return match ? match[1] : null;
	}

	function onPlayerReady(event) {
		fetchSubtitlesAndQuestions();
	}

	function onPlayerStateChange(event) {
		if (event.data === YT.PlayerState.PLAYING) {
			startQuestionChecking();
		} else if (event.data === YT.PlayerState.CUED) {
			fetchSubtitlesAndQuestions();
		}
	}

	async function fetchSubtitlesAndQuestions() {
		if (videoId) {
			const response = await fetch(
				"http://localhost:5000/api/subtitles-questions",
				{
					method: "POST",
					headers: {
						"Content-Type": "application/json",
					},
					body: JSON.stringify({ videoId }),
				},
			);
			const data = await response.json();
			questions = JSON.parse(data.questions).questions;
			transcript = data.transcript;
			currentQuestionIndex = 0;
		}
	}

	async function sendUserResponse(userResponse) {
		question = questions[currentQuestionIndex - 1].question;
		const response = await fetch("http://localhost:5000/api/feedback", {
			method: "POST",
			headers: {
				"Content-Type": "application/json",
			},
			body: JSON.stringify({ userResponse, question, transcript }),
		});
		const feedback = await response.json();
		// Display the feedback to the user
		document.getElementById("feedback").innerHTML = feedback;
		// Resume video playback
		player.playVideo();
	}

	function startQuestionChecking() {
		const checkQuestionInterval = setInterval(() => {
			const currentTime = player.getCurrentTime();
			const currentQuestion = questions[currentQuestionIndex];
			if (currentQuestion && currentTime >= currentQuestion.time) {
				player.pauseVideo();
				question = currentQuestion.question;
				displayQuestion(question);
				clearInterval(checkQuestionInterval);
				currentQuestionIndex++;
			}
		}, 1000);
	}

	function displayQuestion(question) {
		document.getElementById("question").innerHTML = question;
	}
</script>

<main>
	<div class="course-container">
		<div class="course-sidebar">
			<h3>Interactive YouTube Video</h3>
			<ul>
				<li><a href="#video">Video: YouTube Video</a></li>
				<li><a href="#questions">Questions</a></li>
				<li><a href="#feedback">Feedback</a></li>
			</ul>
		</div>

		<div class="course-content">

			<div class="video-section">
				<Card>
					<div class="video-container">
						<div id="player"></div>
					</div>
					<div class="input-container">
						<Textfield
							bind:value={videoUrl}
							label="Enter YouTube video URL"
						/>
						<Button variant="raised" on:click={loadVideo}
							>Load Video</Button
						>
					</div>
				</Card>
			</div>

			<div class="question-section" id="questions">
				<h2>Questions</h2>
				<Card>
					<div class="question-container">
						<p id="question"></p>
						<Textfield
							bind:value={userResponse}
							label="Enter your response"
						/>
						<Button
							variant="raised"
							color="secondary"
							on:click={sendUserResponse}>Submit</Button
						>
					</div>
				</Card>
			</div>

			<div class="feedback-section" id="feedback">
				<h2>Feedback</h2>
				<Card>
					<div class="feedback-container">
						<p id="feedback"></p>
					</div>
				</Card>
			</div>
		</div>
	</div>
	<script src="https://www.youtube.com/iframe_api"></script>
</main>

<style>
	.course-container {
		display: flex;
	}

	.course-sidebar {
		width: 25%;
		padding: 20px;
		background-color: #f2f2f2;
	}

	.course-sidebar h3 {
		margin-top: 0;
	}

	.course-sidebar ul {
		list-style-type: none;
		padding: 0;
	}

	.course-sidebar li {
		margin-bottom: 10px;
	}

	.course-content {
		width: 75%;
		padding: 20px;
	}

	.course-content h1 {
		margin-top: 0;
	}

	.course-content ul {
		list-style-type: disc;
		margin-left: 20px;
	}

	.video-section,
	.question-section,
	.feedback-section {
		margin-bottom: 40px;
	}

	.video-container {
		position: relative;
		padding-bottom: 56.25%;
		height: 0;
		margin-bottom: 20px;
	}

	#player {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	/* ... (additional styles for input container, question container, etc.) */
</style>
