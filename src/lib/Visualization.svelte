<script>
    import { onMount } from "svelte";
    import { createEventDispatcher } from "svelte";
    import * as d3 from "d3";

    const dispatch = createEventDispatcher();

    export let selected_country_details;
    let details_width;
    let details_height;

    function closeVisualization() {
        dispatch("close");
    }

    $: if (selected_country_details) {
        d3.select("#peace").html(selected_country_details.peace_process_text);
        // d3.select("#year").html(individual_info.properties.year);
        // d3.select("#month").html(individual_info.properties.month);
    }
</script>

<div
    class="visualization"
    bind:clientWidth={details_width}
    bind:clientHeight={details_height}
>
    <button class="btn close" on:click={closeVisualization}
        ><i class="fa fa-close"></i></button
    >

    <div id="peace_title_div">
        <h3>Peace Process</h3>
    </div>

    <div id="peace_content">
        <pre id="peace"></pre>
        <br />
        <pre id="year"></pre>
        <br />
        <pre id="month"></pre>
    </div>
</div>

<style>
    .visualization {
        color: white;
        position: fixed;
        right: -100%;
        bottom: 4px;
        width: 500px;
        height: calc(100% - 47px);
        transition: right 0.3s ease;
        background: rgba(32, 32, 32, 0.9);
        box-shadow: -2px 0 5px rgba(0, 0, 0, 0.5);
        border-radius: 3px;
        overflow: hidden;
        z-index: 5;
        font-family: "Montserrat";
        display: flex;
        flex-direction: column;
    }

    @media (max-width: 1450px) {
        .visualization {
            width: 450px;
            height: calc(100% - 41px);
            bottom: 3px;
        }
    }

    @media (max-width: 1200px) {
        .visualization {
            width: 400px;
            height: calc(100% - 35px);
            bottom: 2px;
        }
    }

    @media (max-width: 768px) {
        .visualization {
            width: 100%;
            height: calc(100% - 32px);
            bottom: 2px;
            font-size: 0.8em;
        }
    }

    #peace_content {
        flex: 1;
        height: calc(50% - 40px);
        align-items: center;
        position: relative;
        overflow-y: auto;
        font-size: 14px;
        background-color: none;
        margin-left: 5px;
        margin-right: 5px;
        border-radius: 2px;
    }

    #peace_title_div {
        text-align: center;
        position: relative;
        border-radius: 5px;
    }

    .close {
        position: absolute;
        z-index: 5;
        top: 2px;
        left: 4px;
        background: #fdd900;
        color: black;
        border: none;
        padding: 2px 12px;
        border-radius: 2px;
        font-size: 20px;
        cursor: pointer;
        font-family: "Montserrat";
    }

    h3 {
        color: white;
        margin: auto;
        font-size: 1.2em;
        padding: 3px;
    }

    ul {
        padding: 10px;
    }

    @media only screen and (max-width: 1450px) {
        h3 {
            font-size: 1.1em;
        }
    }

    @media only screen and (max-width: 1200px) {
        h3 {
            font-size: 1em;
        }
    }

    @media only screen and (max-width: 768px) {
        h3 {
            font-size: 0.9em;
        }
    }

    #chart {
        flex: 1;
        display: flex;
        background-color: #171d26;
        height: calc(50% - 40px);
        margin-left: 5px;
        margin-right: 5px;
        margin-bottom: 5px;
        border-radius: 2px;
    }

    /* width */
    #peace_content::-webkit-scrollbar {
        width: 10px;
    }

    /* Track */
    #peace_content::-webkit-scrollbar-track {
        background: #f1f1f1;
    }

    /* Handle */

    #peace_content::-webkit-scrollbar-thumb {
        background: #888;
    }

    /* Handle on hover */
    #peace_content::-webkit-scrollbar-thumb:hover {
        background: #555;
    }

    .btn {
        border: none;
        box-shadow: none;
        border-radius: 2px;
        font-size: 1.1em;
        font-family: "Montserrat";
        color: black;
        padding: 0.2em 1em;
        cursor: pointer;
        background: #fdd92f;
        box-shadow: 0 1px 5px rgba(0, 0, 0, 0.5);
    }

    @media only screen and (max-width: 1450px) {
        .btn {
            font-size: 1em;
        }
    }

    @media only screen and (max-width: 1024px) {
        .btn {
            font-size: 0.9em;
        }
    }

    pre {
        margin-top: 2px;
        margin-bottom: 2px;
        font-family: "Montserrat";
        direction: ltr;
        text-align: left;
        padding-left: 15px;
        padding-right: 15px;
        font-size: 0.9em;
        white-space: pre-wrap;
    }

    @media only screen and (max-width: 1450px) {
        pre {
            font-size: 0.75em;
        }
    }

    @media only screen and (max-width: 1024px) {
        pre {
            font-size: 0.63em;
        }
    }

    :global(a) {
        color: yellow;
        cursor: pointer;
    }

    :global(ul) {
        padding: 10px;
    }
</style>
