<script>
    import ioClient from "socket.io-client";
    import {page} from "$app/stores";

    $: connectionUrl = $page.params?.socketUrl || "";
    $: io = null;

    $: connected = false;
    function socketInit() {
        if(io){
            io.disconnect()
            io.removeAllListeners()
        }
        io = ioClient(connectionUrl)
        io.on("connect", () => {
            connected = true
        })
        io.on("disconnect", () => {
            connected = false
        })
        io.onAny((event, data) => {
            receiveMessage(event, data)
        })


    }
    $: messages = [];

    function receiveMessage(eventName, message) {
        messages = messages.concat({eventName, message});
    }

    $: event = "";
    $: message = "";
    function sendMessage(){
        if(io && connected){
            io.emit(event, message)
        }else{
            alert("Socket not connected")
        }
    }
</script>

<div style="width: 100vw; height: 100vh; display: flex; flex-direction: column; gap: 1rem;">
    <div style="display: flex; gap: 1rem;">
        <input style="width: 30rem;" type="text" id="socketUrl" placeholder="Socket connection url:port" bind:value={connectionUrl}>
        <button id="connect" on:click={socketInit}>Connect</button>
        <span class={connected ? "connected" : "disconnected"}>{connected ? "Connected" : "Disconnected"}</span>
    </div>
    <div style="display: flex; gap: 1rem;">
        <input type="text" placeholder="Event name" bind:value={event}>
        <input type="text" style="width: 20rem;" placeholder="Message" bind:value={message}>
        <button on:click={sendMessage}>Emit</button>
    </div>

    <ul>
        {#each messages as message}
            <li>Event: {message.eventName} - {message.message}</li>
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
