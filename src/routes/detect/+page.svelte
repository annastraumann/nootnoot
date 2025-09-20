<script>
	import ObjectDetection from '$lib/ObjectDetection.svelte';
</script>

<svelte:head>
	<title>Object Detection - NootNoot</title>
</svelte:head>

<div class="page">
	<ObjectDetection />
	
	<div class="info">
		<h3>How it works:</h3>
		<ul>
			<li>📹 <strong>Webcam mode:</strong> Uses your camera for live detection</li>
			<li>📁 <strong>Upload mode:</strong> Analyze your own video files</li>
			<li>🧠 AI model detects objects in real-time</li>
			<li>📦 Draws green boxes around detected objects</li>
			<li>📊 Shows confidence percentage for each detection</li>
		</ul>
		
		<p class="note">
			<strong>Tips:</strong> 
			• For webcam: Allow camera access when prompted<br>
			• For video upload: MP4, WebM, AVI files work best<br>
			• First load might take a few seconds for the AI model
		</p>
	</div>
</div>

<style>
	.page {
		max-width: 800px;
		margin: 0 auto;
		padding: 2rem;
	}
	
	.info {
		margin-top: 2rem;
		padding: 1rem;
		background: #f8f9fa;
		border-radius: 8px;
	}
	
	.info ul {
		margin: 1rem 0;
	}
	
	.info li {
		margin: 0.5rem 0;
	}
	
	.note {
		margin-top: 1rem;
		padding: 1rem;
		background: #fff3cd;
		border: 1px solid #ffeaa7;
		border-radius: 4px;
		color: #856404;
	}
</style>