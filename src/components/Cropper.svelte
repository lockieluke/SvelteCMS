<script lang="ts">
	import Icon from '@iconify/svelte';
	export let image: any = '';
	export let rotate: number | string = 0;
	export let crop_left: object = { initialValue: 10, value: 10 };
	export let crop_right: object = { initialValue: 10, value: 10 };
	export let crop_top: object = { initialValue: 10, value: 10 };
	export let crop_bottom: object = { initialValue: 10, value: 10 };
	export let blurs: any = [];
	export let center: any = { x: 200, y: 115 };
	let currentIndex = 0;
	let blurCoords: object = {
		pageX: 0,
		pageY: 0
	};
	export let SCALE = 1;
	let TR_X = 0;
	let TR_Y = 0;
	export let rotateDetails = {
		scale: 1,
		tr_x: 0,
		tr_y: 0
	};
	const isActive = false;
	let WHOLE_WIDTH = 400,
		WHOLE_HEIGHT = 225;
	let MOUSEDOWN_CORNER = false;
	let MOUSEDOWN_WHOLE = false;
	let CHANGE_LEFT = false,
		CHANGE_RIGHT = false,
		CHANGE_TOP = false,
		CHANGE_BOTTOM = false;
	let MOUSE_START_LEFT = 0,
		MOUSE_START_TOP = 0;
	$: CONT_WIDTH = WHOLE_WIDTH ? WHOLE_WIDTH + 'px' : '400px';
	$: CONT_HEIGHT = WHOLE_HEIGHT ? WHOLE_HEIGHT + 'px' : 'auto';
	let cropping = false;
	let checker;

	function handleMouseDown(e) {
		e.preventDefault();
		WHOLE_WIDTH = document.getElementById('image_handler').offsetWidth;
		WHOLE_HEIGHT = document.getElementById('image_handler').offsetHeight;
		const corner = e.target.closest('.corner');
		const is_whole = e.target.closest('.inner');
		if (corner) {
			MOUSEDOWN_CORNER = true;
			CHANGE_LEFT = corner.dataset.left;
			CHANGE_RIGHT = corner.dataset.right;
			CHANGE_TOP = corner.dataset.top;
			CHANGE_BOTTOM = corner.dataset.bottom;
			MOUSE_START_LEFT = e.pageX;
			MOUSE_START_TOP = e.pageY;
		} else if (is_whole) {
			MOUSEDOWN_WHOLE = true;
			MOUSE_START_LEFT = e.pageX;
			MOUSE_START_TOP = e.pageY;
		}
		return true;
	}
	$: {
		rotateDetails.scale =
			(WHOLE_HEIGHT * Math.abs(Math.sin((90 - rotate) * (Math.PI / 180))) +
				WHOLE_WIDTH * Math.abs(Math.cos((90 - rotate) * (Math.PI / 180)))) /
			WHOLE_HEIGHT;
		rotateDetails.tr_y = -(WHOLE_HEIGHT * Math.sin(rotate * (Math.PI / 180))) / rotateDetails.scale;
	}
	function handleMouseMove(e) {
		if (MOUSEDOWN_CORNER) {
			if (CHANGE_LEFT) {
				checker = crop_left.initialValue + e.pageX - MOUSE_START_LEFT;
				if (checker < 10) {
					checker = 10;
				} else if (checker > WHOLE_WIDTH - crop_right.value - 20) {
					checker = WHOLE_WIDTH - crop_right.value - 20;
				}
				crop_left.value = checker;
			}
			if (CHANGE_RIGHT) {
				checker = crop_right.initialValue + MOUSE_START_LEFT - e.pageX;
				if (checker < 10) {
					checker = 10;
				} else if (checker > WHOLE_WIDTH - crop_left.value - 20) {
					checker = WHOLE_WIDTH - crop_left.value - 20;
				}
				crop_right.value = checker;
			}
			if (CHANGE_TOP) {
				checker = crop_top.initialValue + e.pageY - MOUSE_START_TOP;
				if (checker < 10) {
					checker = 10;
				} else if (checker > WHOLE_HEIGHT - crop_bottom.value - 20) {
					checker = WHOLE_HEIGHT - crop_bottom.value - 20;
				}
				crop_top.value = checker;
			}
			if (CHANGE_BOTTOM) {
				checker = crop_bottom.initialValue + MOUSE_START_TOP - e.pageY;
				if (checker < 10) {
					checker = 10;
				} else if (checker > WHOLE_HEIGHT - crop_top.value - 20) {
					checker = WHOLE_HEIGHT - crop_top.value - 20;
				}
				crop_bottom.value = checker;
			}
		} else if (MOUSEDOWN_WHOLE) {
			let left_checker = crop_left.initialValue + e.pageX - MOUSE_START_LEFT;
			let right_checker = crop_right.initialValue + MOUSE_START_LEFT - e.pageX;
			let top_checker = crop_top.initialValue + e.pageY - MOUSE_START_TOP;
			let bottom_checker = crop_bottom.initialValue + MOUSE_START_TOP - e.pageY;
			if (top_checker < 10 || bottom_checker < 10) {
				if (top_checker < 10) {
					bottom_checker = bottom_checker + top_checker - 10;
					top_checker = 10;
				} else {
					top_checker = top_checker + bottom_checker - 10;
					bottom_checker = 10;
				}
			}

			if (left_checker < 10 || right_checker < 10) {
				if (left_checker < 10) {
					right_checker = right_checker + left_checker - 10;
					left_checker = 10;
				} else {
					left_checker = left_checker + right_checker - 10;
					right_checker = 10;
				}
			}

			crop_top.value = top_checker;
			crop_bottom.value = bottom_checker;
			crop_left.value = left_checker;
			crop_right.value = right_checker;
		}
	}
	function handleMouseUp(e) {
		MOUSEDOWN_WHOLE = false;
		MOUSEDOWN_CORNER = false;
		crop_left.initialValue = crop_left.value;
		crop_right.initialValue = crop_right.value;
		crop_top.initialValue = crop_top.value;
		crop_bottom.initialValue = crop_bottom.value;
	}

	function openCrop() {
		document.addEventListener('mousedown', handleMouseDown);

		document.addEventListener('mousemove', handleMouseMove);

		document.addEventListener('mouseup', handleMouseUp);

		cropping = !cropping;
		if (cropping) {
			SCALE = 1;
			TR_X = 0;
			TR_Y = 0;
			WHOLE_HEIGHT = 0;
		}
	}
	function cropImage() {
		let actual_width = WHOLE_WIDTH - crop_left.value - crop_right.value;
		let actual_height = WHOLE_HEIGHT - crop_top.value - crop_bottom.value;
		let scaleX = WHOLE_WIDTH / actual_width;
		WHOLE_HEIGHT = (400 * actual_height) / actual_width;
		TR_X = -crop_left.value * scaleX;
		TR_Y = -crop_top.value * scaleX;
		SCALE = scaleX;
		cropping = false;
		document.removeEventListener('mousedown', handleMouseDown);
		document.removeEventListener('mousemove', handleMouseMove);
		document.removeEventListener('mouseup', handleMouseUp);
	}

	function mouseupBlurHandler(e) {
		if (blurs[currentIndex]) {
			e.target.removeEventListener('mousemove', moveBlurHandler);
			e.target.removeEventListener('mouseup', handleBlurMouseDown);
			blurs[currentIndex].top.initialValue = blurs[currentIndex].top.value;
			blurs[currentIndex].left.initialValue = blurs[currentIndex].left.value;
			currentIndex = -1;
		}
	}
	function mouseoutBlurHandler(e) {
		e.target.removeEventListener('mousemove', moveBlurHandler);
		e.target.removeEventListener('mouseup', handleBlurMouseDown);
		e.target.removeEventListener('mousedown', mouseupBlurHandler);
		currentIndex = -1;
	}
	function moveBlurHandler(e) {
		blurs[currentIndex].top.value =
			blurs[currentIndex].top.initialValue + (e.pageY - blurCoords.pageY);
		blurs[currentIndex].left.value =
			blurs[currentIndex].left.initialValue + (e.pageX - blurCoords.pageX);
	}
	function handleBlurMouseDown(e) {
		if (!e.target.getAttribute('data')) {
			blurCoords.pageX = e.pageX;
			blurCoords.pageY = e.pageY;
			currentIndex = e.target.getAttribute('key');
			blurs[currentIndex].top.initialValue = blurs[currentIndex].top.value;
			blurs[currentIndex].left.initialValue = blurs[currentIndex].left.value;
			e.target.addEventListener('mousemove', moveBlurHandler);
			e.target.addEventListener('mouseup', mouseupBlurHandler);
			// e.target.addEventListener('mouseout', mouseoutBlurHandler);
		} else {
			blurCoords.pageX = e.pageX;
			blurCoords.pageY = e.pageY;
			currentIndex = e.target.getAttribute('data_key');
			blurs[currentIndex].top.initialValue = blurs[currentIndex].top.value;
			blurs[currentIndex].left.initialValue = blurs[currentIndex].left.value;
			e.target.addEventListener('mousemove', moveCornerHandler);
			e.target.addEventListener('mouseup', mouseupCornerHandler);
			e.target.addEventListener('mouseout', mouseoutCornerHandler);
		}
	}
	function blurCrop() {
		blurs[blurs.length] = {
			top: { initialValue: 0, value: 0 },
			left: { initialValue: 0, value: 0 },
			width: { initialValue: 100, value: 100 },
			height: { initialValue: 100, value: 100 }
		};
	}

	function changeCenter(e) {
		TR_X -= e.layerX - WHOLE_WIDTH / 2;
		TR_Y -= e.layerY - WHOLE_HEIGHT / 2;
		console.log(TR_X, TR_Y);
		e.target.removeEventListener('click', changeCenter);
		e.target.classList.remove('cursor-crosshair');
	}
	function newCenter() {
		const editor = document.getElementById('image_handler');
		editor?.classList.add('cursor-crosshair');
		editor?.addEventListener('click', changeCenter);
	}
	function moveCornerHandler(e) {
		if (e.target.getAttribute('data') === '1') {
			blurs[currentIndex].top.value =
				blurs[currentIndex].top.initialValue + (e.pageY - blurCoords.pageY);
			blurs[currentIndex].height.value =
				blurs[currentIndex].height.initialValue + -(e.pageY - blurCoords.pageY);
			blurs[currentIndex].width.value =
				blurs[currentIndex].width.initialValue + -(e.pageX - blurCoords.pageX);
			blurs[currentIndex].left.value =
				blurs[currentIndex].left.initialValue + (e.pageX - blurCoords.pageX);
		} else if (e.target.getAttribute('data') === '2') {
			blurs[currentIndex].top.value =
				blurs[currentIndex].top.initialValue + (e.pageY - blurCoords.pageY);
			blurs[currentIndex].height.value =
				blurs[currentIndex].height.initialValue + -(e.pageY - blurCoords.pageY);
			blurs[currentIndex].width.value =
				blurs[currentIndex].width.initialValue + (e.pageX - blurCoords.pageX);
		} else if (e.target.getAttribute('data') === '3') {
			blurs[currentIndex].height.value =
				blurs[currentIndex].height.initialValue + (e.pageY - blurCoords.pageY);
			blurs[currentIndex].width.value =
				blurs[currentIndex].width.initialValue + (e.pageX - blurCoords.pageX);
		} else {
			blurs[currentIndex].height.value =
				blurs[currentIndex].height.initialValue + (e.pageY - blurCoords.pageY);
			blurs[currentIndex].width.value =
				blurs[currentIndex].width.initialValue + -(e.pageX - blurCoords.pageX);
			blurs[currentIndex].left.value =
				blurs[currentIndex].left.initialValue + (e.pageX - blurCoords.pageX);
		}
	}
	function mouseupCornerHandler(e) {
		e.target.removeEventListener('mousemove', moveCornerHandler);
		e.target.removeEventListener('mouseup', mouseupCornerHandler);
		e.target.removeEventListener('mouseout', mouseoutCornerHandler);
		blurs[currentIndex].width.initialValue = blurs[currentIndex].width.value;
		blurs[currentIndex].height.initialValue = blurs[currentIndex].height.value;
		currentIndex = -1;
	}

	function mouseoutCornerHandler(e) {
		e.target.removeEventListener('mousemove', moveCornerHandler);
		e.target.removeEventListener('mouseup', mouseupCornerHandler);
		e.target.removeEventListener('mouseout', mouseoutCornerHandler);
		blurs[currentIndex].width.initialValue = blurs[currentIndex].width.value;
		blurs[currentIndex].height.initialValue = blurs[currentIndex].height.value;
		currentIndex = -1;
	}
</script>

<div class="container text-black">
	{#if image}
		<div style="padding: 120px 50px; display: flex; gap: 20px;">
			<div style="position: relative;" id="image_handler">
				{#each blurs as item, index}
					<div>
						<div
							class="absolute blur_handler z-10 backdrop-blur activeBlur"
							style="width: {blurs[index].width.value}px; height: {blurs[index].height
								.value}px; top: {blurs[index].top.value}px; left: {blurs[index].left.value}px;"
							key={index}
							on:mousedown={handleBlurMouseDown}
						>
							<div
								class="corner c_1"
								on:mousedown={handleBlurMouseDown}
								data="1"
								data_key={index}
								data-left="1"
								data-top="1"
							/>
							<div
								class="corner c_2"
								on:mousedown={handleBlurMouseDown}
								data="2"
								data_key={index}
								data-top="1"
								data-right="1"
							/>
							<div
								class="corner c_3"
								on:mousedown={handleBlurMouseDown}
								data="3"
								data_key={index}
								data-right="1"
								data-bottom="1"
							/>
							<div
								class="corner c_4"
								on:mousedown={handleBlurMouseDown}
								data="4"
								data_key={index}
								data-bottom="1"
								data-left="1"
							/>
						</div>
					</div>
				{/each}
				{#if cropping}
					<div
						class="crop_window_handler"
						style="left:{crop_left.value}px;
						 right: {crop_right.value}px;
						 top: {crop_top.value}px;
						 bottom:{crop_bottom.value}px"
					>
						<div class="shadow_handler">
							<div class="shadow s_1" />
							<div class="shadow s_2" />
							<div class="shadow s_3" />
							<div class="shadow s_4" />
						</div>
						<div class="lines_handler">
							<div class="line l_1" />
							<div class="line l_2" />
							<div class="line l_3" />
							<div class="line l_4" />
						</div>
						<div class="corners_handler">
							<div class="corner c_1" data-left="1" data-top="1" />
							<div class="corner c_2" data-top="1" data-right="1" />
							<div class="corner c_3" data-right="1" data-bottom="1" />
							<div class="corner c_4" data-bottom="1" data-left="1" />
						</div>
						<div class="inner" />
					</div>
				{/if}
				{#if isActive}
					<div style="position: absolute;" />
				{/if}
				<div
					class="image_container relative "
					id="imagePart"
					style="width: {CONT_WIDTH}; height: {CONT_HEIGHT};"
				>
					<div
						class="image_container relative "
						style="width: {CONT_WIDTH}; height: {CONT_HEIGHT}; transform-origin:  50% 50%; transform: rotate({rotate}deg) scale({rotateDetails.scale})"
					>
						<img
							class="main_image"
							src={URL.createObjectURL(image)}
							alt="image"
							style="transform: translate( {TR_X}px, {TR_Y}px ) scale({SCALE})"
						/>
					</div>
				</div>
			</div>
		</div>
	{/if}
	{#if !cropping}
		<button on:click={openCrop} class="btn btn-primary text-white p-0.5">
			<Icon icon="material-symbols:crop" width="24" />
		</button>
	{:else}
		<button on:click={cropImage} class="btn btn-primary text-white p-0.5">
			<Icon icon="material-symbols:save" width="24" />
		</button>
	{/if}
	<button on:click={blurCrop} class="btn btn-primary text-white p-0.5"> Blur</button>
	<button on:click={newCenter} class="btn btn-primary text-white p-0.5"> center</button>
	<label for="small-range" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white"
		>Rotate</label
	>
	<input
		id="small-range"
		class="w-full h-1 mb-6 bg-gray-200 rounded-lg appearance-none cursor-pointer range-sm dark:bg-gray-700"
		type="range"
		name="rotate"
		min="0"
		max="360"
		bind:value={rotate}
	/>
</div>

<style>
	#image_handler {
		position: relative;
		overflow: hidden;
	}
	#image_handler::after {
		position: absolute;
		content: ' ';
		inset: 0;
	}
	#image_handler .image_container {
		width: 400px;
	}
	.image_container img {
		transition-duration: 0.3s;
		transition-timing-function: linear;
		width: 100%;
		transform-origin: 0 0;
	}
	.crop_window_handler {
		position: absolute;
		box-shadow: inset 0 0 0 1px #fff;
		z-index: 1;
	}
	.crop_window_handler .shadow {
		position: absolute;
		background: rgba(0, 0, 0, 0.5);
		width: 1000px;
		height: 1000px;
		z-index: 0;
	}
	.crop_window_handler .shadow.s_1 {
		left: 100%;
	}
	.crop_window_handler .shadow.s_2 {
		top: 100%;
		right: 0;
	}
	.crop_window_handler .shadow.s_3 {
		right: 100%;
		bottom: 0;
	}
	.crop_window_handler .shadow.s_4 {
		bottom: 100%;
	}
	.crop_window_handler .line {
		position: absolute;
		background: rgba(255, 255, 255, 0.6);
	}
	.crop_window_handler .line.l_1 {
		left: 33.333%;
		top: 0;
		bottom: 0;
		width: 2px;
	}
	.crop_window_handler .line.l_2 {
		left: 66.666%;
		top: 0;
		bottom: 0;
		width: 2px;
	}
	.crop_window_handler .line.l_3 {
		top: 66.666%;
		left: 0;
		right: 0;
		height: 2px;
	}
	.crop_window_handler .line.l_4 {
		top: 33.333%;
		left: 0;
		right: 0;
		height: 2px;
	}

	.corner {
		position: absolute;
		width: 10px;
		height: 10px;
		border-radius: 50%;
		background: #fff;
		z-index: 2;
	}
	.corner.c_1 {
		left: -4px;
		top: -4px;
		cursor: nw-resize;
	}
	.corner.c_2 {
		right: -4px;
		top: -4px;
		cursor: ne-resize;
	}
	.corner.c_3 {
		right: -4px;
		bottom: -4px;
		cursor: se-resize;
	}
	.corner.c_4 {
		left: -4px;
		bottom: -4px;
		cursor: sw-resize;
	}
	.crop_window_handler .inner {
		cursor: all-scroll;
		width: 100%;
		height: 100%;
		position: relative;
	}
	.activeBlur {
		border: 1px solid white;
	}
	.blur_handler .blur_corner {
		position: absolute;
		width: 10px;
		height: 10px;
		border-radius: 50%;
		background: #fff;
		z-index: 2;
	}
</style>
