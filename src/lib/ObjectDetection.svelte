<script>
	import { onMount } from 'svelte';
	import * as tf from '@tensorflow/tfjs';
	import * as cocoSsd from '@tensorflow-models/coco-ssd';

	let videoElement;
	let canvasElement;
	let model;
	let isDetecting = false;
	let detections = [];

	onMount(async () => {
		// Load the COCO-SSD model
		console.log('Loading model...');
		model = await cocoSsd.load();
		console.log('Model loaded!');
		
		// Start webcam
		await startWebcam();
	});

	async function startWebcam() {
		try {
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
	}
</script>

<div class="detection-container">
	<h2>Object Detection with TensorFlow.js</h2>
	
	<div class="video-container">
		<!-- Webcam video -->
		<video 
			bind:this={videoElement}
			width="640" 
			height="480" 
			autoplay 
			muted
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