<script>
  import { getContext, onDestroy, onMount } from "svelte"
  import Canvas from "./Canvas.svelte";

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
  export let showButtonIcon
  export let penWidth
  export let modalTitle
  export let modalActionButtonText
  export let modalBody
  export let saveBackgroundColour
  export let signatureData
  //export let required

  let initalImage
  let currentImage

  let canBeDisplayed
  let errorMessages = [];
  
  //Budibase SDK
  const { styleable, builderStore, notificationStore } = getContext("sdk");

  //Form API
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  let fieldApi
  let fieldState
  let inBuilder = false

  console.log(fieldState);

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

  function setSignatureValue(img, inital){
    fieldApi.setValue(img);
    currentImage = img;
    if (inital){
      initalImage = img;
    }
  }

  $: checkValid(field, formContext, penWidth, width, height, color, background)

  function checkValid(field, formContext, penWidth, width, height, color, background){
    errorMessages = [];
    canBeDisplayed = true;

    if (!formContext){
      errorMessages.push("Form components need to be wrapped in a form");
    }

    if (!field){
      errorMessages.push("Please select a text field");
    }

    if (!penWidth){
      errorMessages.push("Please set a pen size");
    }

    if (!width){
      errorMessages.push("Please define the width of the signature");
    }

    if (!height){
      errorMessages.push("Please define the height of the signature");
    }

    if (!color){
      errorMessages.push("Please define the colour of the pen");
    }

    if (!background){
      errorMessages.push("Please define the background colour");
    }

    if (errorMessages.length > 0){
      canBeDisplayed=false;
    }
  }

  //$: compareImages(required, initalImage, currentImage);

  //need to figure out how to pass validation
  //function compareImages(required, initalImage, currentImage){
  //  if (required){
  //    if (initalImage === currentImage){
  //      notificationStore.actions.warning(
  //        `Please draw your signature`
  //      )
  //    }
  //  }
  //}

  if (($builderStore.inBuilder)){
    inBuilder = true;
  }
</script>
<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !canBeDisplayed}
    <!-- Display error messages when requirements are not defined -->
    <div class="spectrum-InLineAlert spectrum-InLineAlert--notice">
      <div class="spectrum-InLineAlert-header">
        Signature field error
        <svg class="spectrum-Icon spectrum-Icon--sizeM spectrum-InLineAlert-icon" focusable="false" aria-hidden="true">
          <use xlink:href="#spectrum-icon-18-Alert" />
        </svg>
      </div>
      <div class="spectrum-InLineAlert-content">
        <ul>
          {#each errorMessages as message}
            <li>{message}</li>
          {/each}
        </ul>
      </div>
    </div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>
    <div class="spectrum-Form-itemField">
      <Canvas 
        {width}
        {height}
        {color}
        {background}
        {penWidth}
        {saveBackgroundColour}
        {showClearSignatureButton}
        {showButtonIcon}
        {clearSignatureButtonText}
        {setSignatureValue}
        {modalTitle}
        {modalActionButtonText}
        {modalBody}
        {borderOutline}
        {borderColor}
        {borderWidth}
        {inBuilder}
      >
      </Canvas>
    </div>
    {#if fieldState?.error}
      <div class="error">{fieldState.error}</div>
    {/if}
  {/if}
</div>