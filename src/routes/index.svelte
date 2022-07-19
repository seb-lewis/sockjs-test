<script>
    import {page} from "$app/stores";
    import {onMount} from "svelte";

    $: connectionUrl = "";

    $: sockjs = null;

    $: connected = false;
    function socketInit() {
        // eslint-disable-next-line no-undef
        if(sockjs){
            sockjs.close();
        }
        sockjs = new SockJS(connectionUrl);

        sockjs.onopen = () => {
            connected = true
        };
        sockjs.onmessage = ({data}) => {
            receiveMessage(data)
        }
        sockjs.onclose = () => {
            connected = false
        };
    }
    $: messages = [];

    function receiveMessage(message) {
        messages = messages.concat(message);
    }

    $: event = "";
    $: message = "";
    function sendMessage(){
        if(sockjs && connected){
            sockjs.send(message)
        }else{
            alert("Socket not connected")
        }
        message = "";
    }

    onMount(() => {
        connectionUrl = $page.url.searchParams.get("url");
        if(connectionUrl){
            socketInit();
        }
        $page.url.searchParams.delete("url")
    })
</script>
<svelte:head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/sockjs-client/1.6.1/sockjs.min.js"></script>
</svelte:head>

<div style="width: 100vw; height: 100vh; display: flex; flex-direction: column; gap: 1rem;">
    <form style="display: flex; gap: 1rem;" on:submit|preventDefault={socketInit}>
        <input style="width: 30rem;" type="text" id="socketUrl" placeholder="Socket connection url:port" bind:value={connectionUrl}>
        <button type="submit" id="connect">Connect</button>
        <button type="button" on:click={() => {sockjs && sockjs.close()}} disabled={!connected}>Disconnect</button>
        <span class={connected ? "connected" : "disconnected"}>{connected ? "Connected" : "Disconnected"}</span>
    </form>

    <form style="display: flex; gap: 1rem;" on:submit|preventDefault={sendMessage}>
        <input type="text" style="width: 20rem;" placeholder="Message" bind:value={message}>
        <button disabled={!connected}>Send</button>
    </form>

    <ul>
        {#each messages as message}
            <li>{message}</li>
        {/each}
    </ul>
</div>

<style>
    .connected {
        color: green;
    }
    .disconnected {
        color: red;
    }
</style>
