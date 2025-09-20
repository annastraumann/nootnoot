<script>
	import { onMount } from 'svelte';
	import * as tf from '@tensorflow/tfjs';
	import * as cocoSsd from '@tensorflow-models/coco-ssd';

	let videoElement;
	let canvasElement;
	let fileInput;
	let model;
	let isDetecting = false;
	let detections = [];
	let inputMode = 'webcam'; // 'webcam' or 'upload'
	let uploadedFileName = '';

	onMount(async () => {
		// Load the COCO-SSD model
		console.log('Loading model...');
		model = await cocoSsd.load();
		console.log('Model loaded!');
	});

	async function startWebcam() {
		try {
			// Stop any existing detection
			stopDetection();
			
			const stream = await navigator.mediaDevices.getUserMedia({ 
				video: { width: 640, height: 480 } 
			});
			videoElement.srcObject = stream;
			
			videoElement.addEventListener('loadeddata', () => {
				// Start detecting once video is ready
				detectObjects();
			});
		} catch (error) {
			console.error('Error accessing webcam:', error);
		}
	}

	function handleVideoUpload(event) {
		const file = event.target.files[0];
		if (file && file.type.startsWith('video/')) {
			// Stop any existing detection and webcam
			stopDetection();
			
			// Store filename for display
			uploadedFileName = file.name;
			
			// Create URL for the uploaded video
			const videoURL = URL.createObjectURL(file);
			videoElement.srcObject = null; // Clear webcam stream
			videoElement.src = videoURL;
			videoElement.load();
			
			videoElement.addEventListener('loadeddata', () => {
				// Start detecting once video is ready
				detectObjects();
			});
		}
	}

	async function switchInputMode(mode) {
		inputMode = mode;
		stopDetection();
		uploadedFileName = ''; // Clear uploaded filename
		
		if (mode === 'webcam') {
			await startWebcam();
		} else {
			// Clear video source and wait for file upload
			if (videoElement.srcObject) {
				videoElement.srcObject.getTracks().forEach(track => track.stop());
				videoElement.srcObject = null;
			}
			videoElement.src = '';
		}
	}

	async function detectObjects() {
		if (!model || !videoElement) return;
		
		isDetecting = true;
		
		// Detect objects in the video
		const predictions = await model.detect(videoElement);
		detections = predictions;
		
		// Draw bounding boxes
		drawDetections(predictions);
		
		// Keep detecting (creates a loop)
		if (isDetecting) {
			requestAnimationFrame(detectObjects);
		}
	}

	function drawDetections(predictions) {
		const ctx = canvasElement.getContext('2d');
		
		// Clear previous drawings
		ctx.clearRect(0, 0, canvasElement.width, canvasElement.height);
		
		// Draw each detection
		predictions.forEach(prediction => {
			const [x, y, width, height] = prediction.bbox;
			
			// Draw bounding box
			ctx.strokeStyle = '#00ff00';
			ctx.lineWidth = 3;
			ctx.strokeRect(x, y, width, height);
			
			// Draw label
			ctx.fillStyle = '#00ff00';
			ctx.font = '16px Arial';
			ctx.fillText(
				`${prediction.class} (${Math.round(prediction.score * 100)}%)`,
				x,
				y > 20 ? y - 5 : y + 20
			);
		});
	}

	function stopDetection() {
		isDetecting = false;
		if (videoElement.srcObject) {
			videoElement.srcObject.getTracks().forEach(track => track.stop());
		}
		// Also revoke object URL if it was a uploaded file
		if (videoElement.src && videoElement.src.startsWith('blob:')) {
			URL.revokeObjectURL(videoElement.src);
		}
	}
</script>

<div class="detection-container">
	<h2>Object Detection with TensorFlow.js</h2>
	
	<!-- Input mode selection -->
	<div class="input-mode-selector">
		<button 
			class="mode-button" 
			class:active={inputMode === 'webcam'}
			on:click={() => switchInputMode('webcam')}
		>
			📹 Webcam
		</button>
		<button 
			class="mode-button" 
			class:active={inputMode === 'upload'}
			on:click={() => switchInputMode('upload')}
		>
			📁 Upload Video
		</button>
	</div>
	
	<!-- File upload input (only show when upload mode is selected) -->
	{#if inputMode === 'upload'}
		<div class="file-upload">
			<label for="video-file" class="file-upload-label">
				📁 Choose Video File
				<input 
					id="video-file"
					type="file" 
					accept="video/*" 
					bind:this={fileInput}
					on:change={handleVideoUpload}
					style="display: none;"
				>
			</label>
			<p class="upload-hint">Select an MP4, WebM, AVI, or other video file</p>
			{#if uploadedFileName}
				<p class="uploaded-file">✅ Uploaded: {uploadedFileName}</p>
			{/if}
		</div>
	{/if}
	
	<div class="video-container">
		<!-- Video element (webcam or uploaded file) -->
		<video 
			bind:this={videoElement}
			width="640" 
			height="480" 
			autoplay 
			muted
			controls={inputMode === 'upload'}
		></video>
		
		<!-- Canvas for drawing bounding boxes (overlays on video) -->
		<canvas 
			bind:this={canvasElement}
			width="640" 
			height="480"
		></canvas>
	</div>
	
	<div class="controls">
		<button on:click={stopDetection} disabled={!isDetecting}>
			Stop Detection
		</button>
		{#if inputMode === 'webcam'}
			<button on:click={startWebcam} disabled={isDetecting}>
				Start Webcam
			</button>
		{/if}
	</div>
	
	{#if detections.length > 0}
		<div class="detections-list">
			<h3>Detected Objects:</h3>
			{#each detections as detection}
				<div class="detection-item">
					<strong>{detection.class}</strong> - {Math.round(detection.score * 100)}% confidence
				</div>
			{/each}
		</div>
	{/if}
</div>

<style>
	.detection-container {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
		padding: 2rem;
	}
	
	.input-mode-selector {
		display: flex;
		gap: 0.5rem;
		margin-bottom: 1rem;
	}
	
	.mode-button {
		padding: 0.5rem 1rem;
		background: #f8f9fa;
		border: 2px solid #e9ecef;
		border-radius: 4px;
		cursor: pointer;
		transition: all 0.2s;
	}
	
	.mode-button:hover {
		background: #e9ecef;
	}
	
	.mode-button.active {
		background: #007bff;
		border-color: #007bff;
		color: white;
	}
	
	.file-upload {
		text-align: center;
		margin-bottom: 1rem;
	}
	
	.file-upload-label {
		display: inline-block;
		padding: 1rem 2rem;
		background: #007bff;
		color: white;
		border: 2px solid #007bff;
		border-radius: 8px;
		cursor: pointer;
		font-size: 1rem;
		font-weight: 500;
		transition: all 0.2s;
		margin-bottom: 0.5rem;
	}
	
	.file-upload-label:hover {
		background: #0056b3;
		border-color: #0056b3;
		transform: translateY(-1px);
	}
	
	.upload-hint {
		font-size: 0.9rem;
		color: #6c757d;
		margin: 0.5rem 0 0 0;
	}
	
	.uploaded-file {
		font-size: 0.9rem;
		color: #28a745;
		font-weight: 500;
		margin: 0.5rem 0 0 0;
	}
	
	.video-container {
		position: relative;
		border: 2px solid #ccc;
		border-radius: 8px;
		overflow: hidden;
		display: inline-block;
	}
	
	.video-container video {
		display: block;
	}
	
	.video-container canvas {
		position: absolute;
		top: 0;
		left: 0;
		pointer-events: none;
	}
	
	.controls {
		display: flex;
		gap: 1rem;
	}
	
	button {
		padding: 0.5rem 1rem;
		background: #ff4444;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
	}
	
	button:disabled {
		background: #ccc;
		cursor: not-allowed;
	}
	
	.controls button:not(:first-child) {
		background: #28a745;
	}
	
	.detections-list {
		max-width: 640px;
		width: 100%;
	}
	
	.detection-item {
		padding: 0.5rem;
		margin: 0.25rem 0;
		background: #f0f0f0;
		border-radius: 4px;
	}
</style>