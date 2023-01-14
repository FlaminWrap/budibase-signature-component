<script>
  import { getContext, onDestroy, onMount } from "svelte"

  //Input variables
  export let field
  export let label
	export let width
	export let height
	export let color
	export let background
  export let borderOutline
  export let borderColor
  export let borderWidth
  export let clearSignatureButtonText
  export let showClearSignatureButton
  export let clearButtonConfirmText

  //Random UUID to identify the relevant Drawing Canvas
  let canvasID = self.crypto.randomUUID();
  
  //Budibase SDK
  const { styleable } = getContext("sdk");

  //Form API
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  let fieldApi
  let fieldState

  const formApi = formContext?.formApi;
  const labelPos = fieldGroupContext?.labelPosition || "above";
$: formStep = formStepContext ? $formStepContext || 1 : 1;
$: formField = formApi?.registerField(
    field,
    "text",
    0,
    false,
    null,
    formStep
  );

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  //Canvas
	let canvas
	let context
	let isDrawing
	let start
	
	let t, l
	
	onMount(() => {
		context = canvas.getContext('2d')
		context.lineWidth = 3
		
		handleSize()
	})
	
	$: if(context) {
			context.strokeStyle = color
	}
	
	const handleStart = (({ offsetX: x, offsetY: y }) => { 
		if(color === background) {
			context.clearRect(0, 0, width, height)
		} else {
			isDrawing = true
			start = { x, y }
		}
	})
	
	const handleEnd = () => { isDrawing = false; changeData(); }
	const handleMove = (({ offsetX: x1, offsetY: y1 }) => {
		if(!isDrawing) return
		
		const { x, y } = start
		context.beginPath()
		context.moveTo(x, y)
		context.lineTo(x1, y1)
		context.closePath()
		context.stroke()
		
		start = { x: x1, y: y1 }
	})
	
	const handleSize = () => {
		const { top, left } = canvas.getBoundingClientRect()
		t = top
		l = left
	}

	function changeData(){
		let canvasData = document.getElementById(canvasID);
    fillCanvasBackgroundWithColor(canvasData, background)
		let img = canvasData.toDataURL("image/png");
    fieldApi.setValue(img);
  }

  function fillCanvasBackgroundWithColor(canvas, color) {
    // Thank you Caleb Miller @ Stackoverflow
    // Get the 2D drawing context from the provided canvas.
    const context = canvas.getContext('2d');

    // We're going to modify the context state, so it's
    // good practice to save the current state first.
    context.save();

    // Normally when you draw on a canvas, the new drawing
    // covers up any previous drawing it overlaps. This is
    // because the default `globalCompositeOperation` is
    // 'source-over'. By changing this to 'destination-over',
    // our new drawing goes behind the existing drawing. This
    // is desirable so we can fill the background, while leaving
    // the chart and any other existing drawing intact.
    // Learn more about `globalCompositeOperation` here:
    // https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation
    context.globalCompositeOperation = 'destination-over';

    // Fill in the background. We do this by drawing a rectangle
    // filling the entire canvas, using the provided color.
    context.fillStyle = color;
    context.fillRect(0, 0, canvas.width, canvas.height);

    // Restore the original context state from `context.save()`
    context.restore();
  }

  function clearCanvas(){
    const canvas = document.getElementById(canvasID)
    const context = canvas.getContext('2d');
    context.clearRect(0, 0, canvas.width, canvas.height);
  }

  function clearButtonConfirm() {
  let text = clearButtonConfirmText;
  if (confirm(text) == true) {
    clearCanvas()
  }
}
  </script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form</div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>
    <div class="spectrum-Form-itemField">
      {#if showClearSignatureButton}
        <div style="padding-right:5px;padding-bottom:5px;">
          <button on:click={clearButtonConfirm}>
            {clearSignatureButtonText}
          </button>
        </div>
      {/if}
      <canvas style="outline-style: {borderOutline}; outline-color: {borderColor}; outline-width: {borderWidth}"
      id={canvasID}
      {width}
      {height}
      style:background
      bind:this={canvas} 
      on:mousedown={handleStart}	
      on:touchstart={e => {
        const { clientX, clientY } = e.touches[0]
        handleStart({
          offsetX: clientX - l,
          offsetY: clientY - t
        })
      }}	
      on:mouseup={handleEnd}				
      on:touchend={handleEnd}				
      on:mouseleave={handleEnd}
      on:mousemove={handleMove}
      on:touchmove={e => {
        const { clientX, clientY } = e.touches[0]
        handleMove({
          offsetX: clientX - l,
          offsetY: clientY - t
        })
      }}
      />
    </div>
    {#if !field}
    <div class="error">Please select a text field</div>
    {/if}
    {#if !width}
    <div class="error">Please define width e.g. 300</div>
    {/if}
    {#if !height}
    <div class="error">Please define height e.g. 300</div>
    {/if}
    {#if !color}
    <div class="error">Please define a pen colour e.g. Black</div>
    {/if}
    {#if !background}
    <div class="error">Please define a background colour e.g. White</div>
    {/if}
    {#if fieldState?.error}
      <div class="error">{fieldState.error}</div>
    {/if}
  {/if}
</div>

<style>
  .placeholder {
    color: var(--spectrum-global-color-gray-600);
  }
  label {
    white-space: nowrap;
  }
  label.hidden {
    padding: 0;
  }
  .spectrum-Form-itemField {
    position: relative;
    width: 100%;
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
  .error {
    color: var(
      --spectrum-semantic-negative-color-default,
      var(--spectrum-global-color-red-500)
    );
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
</style>