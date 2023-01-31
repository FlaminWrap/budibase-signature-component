<script>
	import { onMount } from 'svelte'
    import Button from "../node_modules/@budibase/bbui/src/Button/Button.svelte"
    import Modal from "../node_modules/@budibase/bbui/src/Modal/Modal.svelte"
    import ModalContent from "../node_modules/@budibase/bbui/src/Modal/ModalContent.svelte"
	
	export let width = 300
	export let height = 300
	export let color = '#333'
	export let background = '#fff'
    export let penWidth = 4
    export let saveBackgroundColour = true
    export let setSignatureValue = () => {}
    export let showClearSignatureButton = true
    export let showButtonIcon = true
    export let clearSignatureButtonText = "Clear Signature"
    export let modalTitle = "Clear Signature"
    export let modalActionButtonText = "Clear"
    export let modalBody = "This will clear the current signature, do you want to contiune?"
    export let borderOutline = "none"
    export let borderColor = "#000000"
    export let borderWidth = "1px"
    export let inBuilder = false
	
    let eraseSignatureModal
	let canvas
	let context
	let isDrawing
	let start
	
	let t, l
	
	onMount(() => {
        if (!inBuilder){
            context = canvas.getContext('2d')
            context.lineWidth = penWidth;
            context.lineJoin = "round";
            context.lineCap = "round";
            
            handleSize();
            saveSignature(true);
        }
	})
	
	$: if(context) {
			context.strokeStyle = color
	}
	
	const handleStart = (({ offsetX: x, offsetY: y }) => { 
        if (!inBuilder){
            if(color === background) {
                context.clearRect(0, 0, width, height)
            } else {
                isDrawing = true
                start = { x, y }
            }
        }
	})
	
	const handleEnd = () => { 
        if (!inBuilder){
            isDrawing = false;
            saveSignature(false);
        }
    }

	const handleMove = (({ offsetX: x1, offsetY: y1 }) => {
        if (!inBuilder){
            if(!isDrawing) return
            
            const { x, y } = start
            context.beginPath()
            context.moveTo(x, y)
            context.lineTo(x1, y1)
            context.closePath()
            context.stroke()
            
            start = { x: x1, y: y1 }
        }
	})
	
	const handleSize = () => {
        const { top, left } = canvas.getBoundingClientRect()
        t = top
        l = left
	}

    function clearCanvas(){
        if (!inBuilder){
            const context = canvas.getContext('2d');
            context.clearRect(0, 0, canvas.width, canvas.height);
            saveSignature(false);
        }
    }

    function fillCanvasBackgroundWithColor(color) {
        if (!inBuilder){
            if (saveBackgroundColour){
                const context = canvas.getContext('2d');
                context.save();
                context.globalCompositeOperation = 'destination-over';
                context.fillStyle = color;
                context.fillRect(0, 0, canvas.width, canvas.height);
                context.restore();
            }
        }
    }

    function saveSignature(inital){
        if (!inBuilder){
            fillCanvasBackgroundWithColor(background);
            let img = canvas.toDataURL("image/png");
            setSignatureValue(img, inital);
        }
    }

</script>

<svelte:window on:resize={handleSize} />
{#if showClearSignatureButton}
<div style="padding-right:8px;padding-bottom:8px;">
    {#if showButtonIcon}
        {#if !inBuilder}
        <Button icon="Erase" primary on:click={eraseSignatureModal.show}>
            {clearSignatureButtonText}
        </Button>
        {:else}
        <Button icon="Erase" primary>
            {clearSignatureButtonText}
        </Button>
        {/if}
    {:else}
        {#if !inBuilder}
            <Button primary on:click={eraseSignatureModal.show}>
                {clearSignatureButtonText}
            </Button>
        {:else}
            <Button primary>
                {clearSignatureButtonText}
            </Button>
        {/if}
    {/if}
    {#if !inBuilder}
    <Modal bind:this={eraseSignatureModal}>
        <ModalContent
            title={modalTitle}
            confirmText={modalActionButtonText}
            onConfirm={clearCanvas}
            >
            <span
                >{modalBody}</span
            >
        </ModalContent>
    </Modal>
    {/if}
</div>
{/if}
<canvas style="outline-style: {borderOutline}; outline-color: {borderColor}; outline-width: {borderWidth}"
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