<script>
    import { onMount } from "svelte";
    import { createEventDispatcher } from "svelte";
    import * as d3 from "d3";

    const dispatch = createEventDispatcher();

    export let selected_country_details;
    let details_width;
    let details_height;
    let overview_height;

    let acled_month, acled_year, ucdp_month, ucdp_year, tracker_link, pax_link;

    $: if (selected_country_details) {
        acled_month = +selected_country_details.acled_events_last_month;
        acled_year = +selected_country_details.acled_events_last_12;

        ucdp_month = +selected_country_details.ucdp_events_last_month;
        ucdp_year = +selected_country_details.ucdp_events_last_12;

        tracker_link = selected_country_details.tracker_url;
        pax_link = selected_country_details.search_pax;
    }

    $: console.log(selected_country_details);

    function closeVisualization() {
        dispatch("close");
    }
</script>

<div
    class="visualization"
    bind:clientWidth={details_width}
    bind:clientHeight={details_height}
>
    <div id="peace_title_div">
        <button class="btn close" on:click={closeVisualization}
            ><i class="fa fa-close"></i></button
        >
        <h3>Peace Process</h3>
    </div>

    <div id="peace_content">
        <div id="overview" bind:clientHeight={overview_height}>
            <h5>Overview</h5>
            <div class="content-wrapper">
                <div class="content-box">
                    <h5>Last Month</h5>
                    <div id="acled_month">ACLED: {acled_month}</div>
                    <div id="ucdp_month">UCDP: {ucdp_month}</div>
                </div>
                <div class="content-box">
                    <h5>Last Year</h5>
                    <div id="acled_year">ACLED: {acled_year}</div>
                    <div id="ucdp_year">UCDP: {ucdp_year}</div>
                </div>
            </div>
        </div>
        <div id="general">
            <h5>General</h5>
            {@html selected_country_details?.general_updates}
        </div>
        <div id="peace_process">
            <h5>Peace Process</h5>
            {@html selected_country_details?.peace_process_text}
        </div>
        <div id="tracker">
            <h5>Tracker</h5>
            <div class="content-wrapper">
                <div class="content-box">
                    <h5>Tracker</h5>
                    <div id="tracker_link">
                        <a href={tracker_link} target="_blank"
                            ><img
                                src="../src/assets/pax.png"
                                alt="pax logo"
                            /></a
                        >
                    </div>
                </div>
                <div class="content-box">
                    <h5>Search PA-X</h5>
                    <div id="pax_link">
                        <a href={pax_link} target="_blank"
                            ><img
                                src="../src/assets/search.png"
                                alt="search icon"
                            /></a
                        >
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<style>
    :root {
        color-scheme: dark;
    }

    .visualization {
        color: white;
        position: fixed;
        right: -100%;
        bottom: 4px;
        width: 500px;
        height: calc(100% - 47px);
        transition: right 0.3s ease;
        background: rgb(0, 0, 0);
        /* box-shadow: -2px 0 5px rgba(0, 0, 0, 0.5); */
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
        flex-grow: 1;
        display: flex;
        flex-direction: column;
        align-items: center;
        position: relative;
        overflow-y: auto;
        font-size: 0.9em;
        margin: 5px;
        border-radius: 2px;
        gap: 4px;
    }

    @media only screen and (max-width: 1450px) {
        #peace_content {
            font-size: 0.8em;
        }
    }

    @media only screen and (max-width: 1200px) {
        #peace_content {
            font-size: 0.8em;
        }
    }

    @media only screen and (max-width: 768px) {
        #peace_content {
            font-size: 0.8em;
        }
    }

    #overview,
    #tracker {
        flex-grow: 1;
        flex-basis: 0;
        width: 100%;
        overflow-y: hidden;
        display: flex;
        flex-direction: column;
    }

    .content-wrapper {
        flex-grow: 1;
        display: flex;
        gap: 5px;
        padding: 5px;
        overflow-y: auto;
        flex-direction: row;
        align-items: stretch;
    }

    .content-box {
        flex-basis: 50%;
        background: #1a1c25;
        color: white;
        padding: 5px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
        display: flex;
        flex-direction: column;
        box-sizing: border-box;
        overflow: hidden; /* Prevent overflow from disrupting layout */
    }

    .content-box h5 {
        margin: 0;
        padding-bottom: 5px;
    }

    #pax_link a,
    #tracker_link a {
        display: block;
        width: 80%;
        height: 80%;
    }

    #pax_link img,
    #tracker_link img {
        width: 100%;
        height: auto; /* Maintain the aspect ratio */
        max-width: 100%; /* Prevent the image from becoming too large */
        max-height: 100%;
        object-fit: contain; /* Ensure the image scales properly without distortion */
    }

    /* Adjusting the flexbox container to be responsive */
    #pax_link,
    #tracker_link {
        flex-grow: 1; /* Allow these elements to take up the remaining space */
        display: flex;
        align-items: center;
        justify-content: center; /* Center the content */
        overflow: hidden;
        cursor: pointer;
        position: relative; /* Position relative to apply the inset shadow */
        box-sizing: border-box; /* Ensure padding/border are included in the width/height */
    }

    /* Hover state for the parent elements */
    #pax_link:hover,
    #tracker_link:hover {
        box-shadow: inset 0 0 10px rgba(255, 255, 255, 0.5); /* Inner shadow on hover */
    }
    #acled_month,
    #acled_year,
    #ucdp_month,
    #ucdp_year {
        flex-grow: 1; /* Allow these elements to take up the remaining space */
        display: flex;
        align-items: center;
        justify-content: center; /* Center text if desired */
        overflow: hidden;
    }

    #general,
    #peace_process {
        flex-grow: 2;
        flex-basis: 0;
        width: 100%;
        overflow-y: auto;
    }

    /* Style for the h5 titles */
    h5 {
        margin: 0;
        padding: 5px 15px;
        background-color: #444444;
        color: white;
        font-size: 1em;
        border-bottom: 1px solid #333;
    }

    @media only screen and (max-width: 768px) {
        h5 {
            font-size: 0.9em;
            padding: 8px 12px;
        }
    }

    #peace_title_div {
        text-align: center;
        justify-content: center;
        position: relative;
        border-radius: 5px;
    }

    #general,
    #peace_process,
    #overview,
    #tracker {
        background: #1a1c25;
    }


    h3 {
        color: white;
        margin: auto;
        font-size: 1.2em;
        padding: 3px;
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

    .btn.close {
        position: absolute;
        left: 4px;
        top: 2px;
        background: none;
        color: #fdd900;
        border: none;
        padding: 2px 12px;
        border-radius: 2px;
        font-size: 20px;
        cursor: pointer;
        font-family: "Montserrat";
        transition: border 0.3s ease;
    }

    .btn.close:hover {
        border: 1px solid #fdd900;
    }

    @media only screen and (max-width: 1450px) {
        .btn.close {
            font-size: 1.1em;
        }
    }

    @media only screen and (max-width: 1024px) {
        .btn.close {
            font-size: 1em;
        }
    }

    :global(pre) {
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
