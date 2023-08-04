<script lang="ts">
    let name: string = 'world';

    function greet(name: string) {
        alert(`Hello, ${name}!`);
    }
</script>

<div>
    <button on:click={() => greet(name)}>Greet {name}</button>
</div>
