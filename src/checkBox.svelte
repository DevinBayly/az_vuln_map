<script>
import {activeProp} from "./store.js"
export let propName
let checkEle
let updateStore = ()=> {
    if (checkEle.checked) {
        activeProp.set(propName)
    }
}
</script>


<label>
{propName}
<input bind:this={checkEle} on:change={updateStore} type="checkbox">
</label>