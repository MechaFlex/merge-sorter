<script lang="ts">
    import { createEventDispatcher } from "svelte"

    export let name: string = ""
    export let uid: string = ""

    const dispatch = createEventDispatcher()
    const onDelete = () => dispatch("removeitem", uid)
</script>

<li>
    <span class="listItemText" id={uid}>{name}</span>
    <span class="deleteItem" on:click={ onDelete }>✖</span>
</li>

<style lang="scss">
    .listItemText {
        position: relative;
        display: block;
        width: calc(100% - 4rem);
        overflow: hidden;
        white-space: nowrap;

        //Adds the fade towards the end if the text is too long.
        &::after {
            position: absolute;
            content: "";
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, rgba(17,17,17,0) 0%, rgba(17,17,17,0) 80%, rgba(17,17,17,1) 100%);
        }
    }

    .deleteItem {
        position: absolute;
        color: inherit;
        top: 1.5rem;
        right: 1.5rem;
        user-select: none;
        width: 3rem;
        height: 3rem;
        text-align: center;
        transition: 0.4s;
        border-radius: 0.5rem;

        &:hover {
            transition: 0.3s;
            background-color: #222;
            color: red;
        }

        &:active {
            transition: 0.1s;
            background-color: #333;
        }
    }
</style>