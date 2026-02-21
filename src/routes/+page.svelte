<script lang="ts">
	import { onMount } from 'svelte';

	let imageUrl = $state<string | null>(null);
	let isDragging = $state(false);
	let dragDepth = 0;
	let fileInput = $state<HTMLInputElement | null>(null);

	const setAppHeight = () => {
		document.documentElement.style.setProperty('--app-height', `${window.innerHeight}px`);
	};

	const preventIOSGesture = (event: Event) => {
		event.preventDefault();
	};

	const preventMultiTouchZoom = (event: TouchEvent) => {
		if (event.touches.length > 1) {
			event.preventDefault();
		}
	};

	let lastTouchEnd = 0;
	const preventDoubleTapZoom = (event: TouchEvent) => {
		const now = Date.now();
		if (now - lastTouchEnd <= 300) {
			event.preventDefault();
		}
		lastTouchEnd = now;
	};

	const clearImage = () => {
		if (imageUrl) {
			URL.revokeObjectURL(imageUrl);
			imageUrl = null;
		}
	};

	const setImageFromFile = (file?: File | null) => {
		if (!file || !file.type.startsWith('image/')) {
			return;
		}

		const nextImageUrl = URL.createObjectURL(file);
		if (imageUrl) {
			URL.revokeObjectURL(imageUrl);
		}
		imageUrl = nextImageUrl;
	};

	const openPicker = () => {
		fileInput?.click();
	};

	const handleFileChange = (event: Event) => {
		const input = event.currentTarget as HTMLInputElement;
		setImageFromFile(input.files?.[0]);
		input.value = '';
	};

	const handleDragEnter = (event: DragEvent) => {
		event.preventDefault();
		dragDepth += 1;
		isDragging = true;
	};

	const handleDragOver = (event: DragEvent) => {
		event.preventDefault();
		isDragging = true;
	};

	const handleDragLeave = (event: DragEvent) => {
		event.preventDefault();
		dragDepth = Math.max(0, dragDepth - 1);
		if (dragDepth === 0) {
			isDragging = false;
		}
	};

	const handleDrop = (event: DragEvent) => {
		event.preventDefault();
		dragDepth = 0;
		isDragging = false;
		setImageFromFile(event.dataTransfer?.files?.[0]);
	};

	onMount(() => {
		setAppHeight();
		window.addEventListener('resize', setAppHeight);
		window.addEventListener('orientationchange', setAppHeight);

		document.addEventListener('gesturestart', preventIOSGesture, { passive: false });
		document.addEventListener('gesturechange', preventIOSGesture, { passive: false });
		document.addEventListener('gestureend', preventIOSGesture, { passive: false });
		document.addEventListener('touchmove', preventMultiTouchZoom, { passive: false });
		document.addEventListener('touchend', preventDoubleTapZoom, { passive: false });

		return () => {
			window.removeEventListener('resize', setAppHeight);
			window.removeEventListener('orientationchange', setAppHeight);
			document.removeEventListener('gesturestart', preventIOSGesture);
			document.removeEventListener('gesturechange', preventIOSGesture);
			document.removeEventListener('gestureend', preventIOSGesture);
			document.removeEventListener('touchmove', preventMultiTouchZoom);
			document.removeEventListener('touchend', preventDoubleTapZoom);
			clearImage();
		};
	});
</script>

<input
	class="hidden-input"
	bind:this={fileInput}
	type="file"
	accept="image/*"
	onchange={handleFileChange}
/>

<main
	class="screen"
	ondragenter={handleDragEnter}
	ondragover={handleDragOver}
	ondragleave={handleDragLeave}
	ondrop={handleDrop}
>
	<section class="safe-area">
		{#if imageUrl}
			<div class="image-frame">
				<img src={imageUrl} alt="" draggable="false" />
			</div>
			<button class="change-image" type="button" onclick={openPicker}>Select image</button>
		{:else}
			<button
				class="dropzone"
				class:dragging={isDragging}
				type="button"
				onclick={openPicker}
				aria-label="Select image or drop image here"
			>
				<span class="dropzone-title">Select image</span>
				<span class="dropzone-subtitle">/ drop image here</span>
			</button>
		{/if}

		{#if isDragging}
			<div class="drop-overlay">Drop image to display</div>
		{/if}
	</section>
</main>

<style>
	:global(:root) {
		--app-height: 100vh;
	}

	:global(html, body) {
		margin: 0;
		width: 100%;
		height: 100%;
		overflow: hidden;
		background: #090909;
		overscroll-behavior: none;
	}

	:global(body) {
		touch-action: none;
	}

	.hidden-input {
		position: absolute;
		width: 1px;
		height: 1px;
		opacity: 0;
		pointer-events: none;
	}

	.screen {
		width: 100vw;
		height: var(--app-height);
		min-height: 100vh;
		min-height: -webkit-fill-available;
		background: #090909;
		overflow: hidden;
		touch-action: none;
	}

	.safe-area {
		position: relative;
		width: 100%;
		height: 100%;
		box-sizing: border-box;
		padding-top: env(safe-area-inset-top);
		padding-right: env(safe-area-inset-right);
		padding-bottom: env(safe-area-inset-bottom);
		padding-left: env(safe-area-inset-left);
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.image-frame {
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		min-width: 0;
		min-height: 0;
	}

	.image-frame img {
		width: 100%;
		height: 100%;
		object-fit: contain;
		object-position: center;
		-webkit-user-drag: none;
		user-select: none;
		pointer-events: none;
	}

	.change-image {
		position: absolute;
		right: 0.75rem;
		bottom: 0.75rem;
		border: 0;
		border-radius: 999px;
		padding: 0.65rem 0.9rem;
		font-size: 0.85rem;
		font-weight: 600;
		background: rgba(30, 30, 30, 0.8);
		color: #f7f7f7;
		backdrop-filter: blur(8px);
		-webkit-backdrop-filter: blur(8px);
	}

	.dropzone {
		width: min(38rem, 92%);
		aspect-ratio: 4 / 3;
		border: 2px dashed rgba(255, 255, 255, 0.28);
		border-radius: 1.25rem;
		background:
			radial-gradient(circle at 15% 20%, rgba(92, 132, 255, 0.22), transparent 55%),
			radial-gradient(circle at 85% 80%, rgba(86, 255, 223, 0.16), transparent 48%),
			rgba(24, 24, 24, 0.7);
		color: #f0f0f0;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 0.35rem;
		text-align: center;
		cursor: pointer;
		padding: 1.5rem;
		box-sizing: border-box;
	}

	.dropzone.dragging {
		border-color: rgba(134, 255, 221, 0.8);
		background:
			radial-gradient(circle at 15% 20%, rgba(92, 132, 255, 0.3), transparent 55%),
			radial-gradient(circle at 85% 80%, rgba(86, 255, 223, 0.28), transparent 48%),
			rgba(24, 24, 24, 0.88);
	}

	.dropzone-title {
		font-size: clamp(1.25rem, 4vw, 1.7rem);
		font-weight: 700;
		letter-spacing: 0.02em;
	}

	.dropzone-subtitle {
		font-size: clamp(1rem, 2.4vw, 1.2rem);
		opacity: 0.8;
	}

	.drop-overlay {
		position: absolute;
		inset: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		background: rgba(10, 10, 10, 0.44);
		color: #f6f6f6;
		font-size: clamp(1rem, 2.8vw, 1.4rem);
		font-weight: 600;
		pointer-events: none;
	}
</style>
