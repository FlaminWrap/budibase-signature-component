<script>
  import { getContext, onDestroy, onMount } from "svelte"
  import Canvas from "./Canvas.svelte";
  import FieldLabel from "../node_modules/@budibase/bbui/src/Form/FieldLabel.svelte"
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
  export let validation
  export let tooltip = ""
  export let buttonType
  export let buttonSize
  export let buttonQuiet
  export let signatureLine
  export let xOnSignatureLine
  export let xType
  export let signatureLineThickness
  export let xSize
  export let signatureLineColor

  let initalImage
  let currentImage

  let buttonStyle = []
  let buttonIcon

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

  const formApi = formContext?.formApi;
  const labelPos = fieldGroupContext?.labelPosition || "above";
$: formStep = formStepContext ? $formStepContext || 1 : 1;
$: formField = formApi?.registerField(
    field,
    "text",
    "",
    false,
    validation,
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
    if (initalImage === currentImage){
      fieldApi.setValue("");
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

  if (($builderStore.inBuilder)){
    inBuilder = true;
  }

  $: setupButtonType(buttonType)

  function setupButtonType(buttonType){
    buttonStyle = []
    for (let i = 0; i < 5; i++){
      if (buttonType === i){
        buttonStyle.push(true)
      } else {
        buttonStyle.push(false)
      }
    }
  }

  $: buttonIconSetup(showButtonIcon)

  function buttonIconSetup(showButtonIcon){
    if (showButtonIcon){
      buttonIcon = "Erase"
    } else {
      buttonIcon = ""
    }
  }

  $: borderWidthEnabled(borderOutline)

  function borderWidthEnabled(borderOutline){
    if (borderOutline === "none"){
      borderWidth = "0px"
    }
  }
</script>

<div class="spectrum-Form-item" class:above={labelPos === "above"} use:styleable={$component.styles}>
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
  {#if label}
    <FieldLabel forId={fieldState?.fieldId} {label} position={labelPos} {tooltip} />
  {/if}
    <div class="spectrum-Form-itemField">
      <Canvas
        {width}
        {height}
        {color}
        {background}
        {penWidth}
        {saveBackgroundColour}
        {showClearSignatureButton}
        {buttonIcon}
        {clearSignatureButtonText}
        {setSignatureValue}
        {modalTitle}
        {modalActionButtonText}
        {modalBody}
        {borderOutline}
        {borderColor}
        {borderWidth}
        {inBuilder}
        {buttonStyle}
        {buttonSize}
        {buttonQuiet}
        {signatureLine}
        {xOnSignatureLine}
        {xType}
        {signatureLineThickness}
        {xSize}
        {signatureLineColor}
      >
      </Canvas>

    </div>
    {#if fieldState?.error}
      <div class="error">{fieldState.error}</div>
    {/if}
  {/if}
</div>

<style>
  .spectrum-Form-item.above {
    display: flex;
    flex-direction: column;
  }
  .spectrum-Form-itemField {
    position: relative;
    width: 100%;
  }

  .error {
    color: var(
      --spectrum-semantic-negative-color-default,
      var(--spectrum-global-color-red-500)
    );
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }
</style>
