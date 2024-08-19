<script>
    import { onMount } from "svelte";
    import { createEventDispatcher } from "svelte";
    import { faArrowUp, faArrowDown } from "@fortawesome/free-solid-svg-icons";
    import * as d3 from "d3";
    import { FontAwesomeIcon } from "@fortawesome/svelte-fontawesome";

    const dispatch = createEventDispatcher();

    export let selected_country_details;
    let details_width;
    let details_height;
    let overview_height;
    let vis_width = 100;

    $: gpi_scaling = d3
        .scaleLinear()
        .range([5, vis_width - 5])
        .domain([1, 163]);
    $: cpi_scaling = d3
        .scaleLinear()
        .range([5, vis_width - 5])
        .domain([1, 181]);

    let acled_month,
        acled_month_change,
        acled_year,
        acled_year_change,
        ucdp_month,
        ucdp_month_change,
        ucdp_year,
        ucdp_year_change,
        tracker_link,
        pax_link,
        gpi,
        cpi;

    $: if (selected_country_details) {
        //populate acled data
        acled_month = +selected_country_details.acled_fatalities_last_month;
        acled_month_change =
            +selected_country_details.acled_change_in_fatalities_last_month;
        acled_year = +selected_country_details.acled_fatalities_last_12;
        acled_year_change =
            +selected_country_details.acled_change_in_fatalities_last_12_months;

        gpi = +selected_country_details.gpi_rank_value;
        cpi = +selected_country_details.cpi_rank_value;

        //populate ucdp data
        ucdp_month = +selected_country_details.ucdp_fatalities_last_month;
        ucdp_month_change =
            +selected_country_details.ucdp_change_in_fatalities_last_month;
        ucdp_year = +selected_country_details.ucdp_fatalities_last_12;
        ucdp_year_change =
            +selected_country_details.ucdp_change_in_fatalities_last_12_months;

        //populate links to pax and tracker
        tracker_link = selected_country_details.tracker_url;
        if (tracker_link == "") {
            tracker_link = "https://pax.peaceagreements.org/tracker/all/";
        }
        pax_link = selected_country_details.search_pax;
    }

    $: console.log(selected_country_details, tracker_link);

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
                    <h5 style="background-color: #2a2a2a">
                        Fatalities Last Month
                    </h5>
                    <div class="row">
                        <div id="acled_month">ACLED: {acled_month}</div>
                        <div id="acled_m_change" class="tooltip-container">
                            {#if acled_month_change > 0}
                                <FontAwesomeIcon icon={faArrowUp} />
                            {:else if acled_month_change < 0}
                                <FontAwesomeIcon icon={faArrowDown} />
                            {/if}
                            {acled_month_change}
                            <span class="tooltip-text"
                                >Change from <br /> last month</span
                            >
                        </div>
                    </div>
                    <div class="row">
                        <div id="ucdp_month">UCDP: {ucdp_month}</div>
                        <div id="ucdp_m_change" class="tooltip-container">
                            {#if ucdp_month_change > 0}
                                <FontAwesomeIcon icon={faArrowUp} />
                            {:else if ucdp_month_change < 0}
                                <FontAwesomeIcon icon={faArrowDown} />
                            {/if}
                            {ucdp_month_change}
                            <span class="tooltip-text"
                                >Change from <br /> last month</span
                            >
                        </div>
                    </div>
                </div>
                <div class="content-box">
                    <h5 style="background-color: #2a2a2a">
                        Fatalities Last Year
                    </h5>
                    <div class="row">
                        <div id="acled_year">ACLED: {acled_year}</div>
                        <div id="acled_y_change" class="tooltip-container">
                            {#if acled_year_change > 0}
                                <FontAwesomeIcon icon={faArrowUp} />
                            {:else if ucdp_month_change < 0}
                                <FontAwesomeIcon icon={faArrowDown} />
                            {/if}
                            {acled_year_change}
                            <span class="tooltip-text"
                                >Change from <br /> last year</span
                            >
                        </div>
                    </div>
                    <div class="row">
                        <div id="ucdp_year">UCDP: {ucdp_year}</div>
                        <div id="ucdp_y_change" class="tooltip-container">
                            {#if ucdp_year_change > 0}
                                <FontAwesomeIcon icon={faArrowUp} />
                            {:else if ucdp_month_change < 0}
                                <FontAwesomeIcon icon={faArrowDown} />
                            {/if}
                            <span class="tooltip-text"
                                >Change from <br /> last year</span
                            >
                            {ucdp_year_change}
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div id="general">
            <h5>General</h5>
            <div class="scrollable-content">
                <p style="margin-bottom: 5px;">Global Peace Index Ranking:</p>
                <div id="gpi" bind:clientWidth={vis_width}>
                    <svg height="45px" width={vis_width}>
                        <defs>
                            <linearGradient id="Gradient1">
                                <stop class="stop1" offset="0%" />
                                <stop class="stop2" offset="50%" />
                                <stop class="stop3" offset="100%" />
                            </linearGradient>
                        </defs>
                        <rect
                            id="rect1"
                            x="5"
                            y="0"
                            rx="2"
                            width={vis_width - 5}
                            height="30"
                        />
                        <line
                            x1={gpi_scaling(gpi)}
                            y1="0"
                            x2={gpi_scaling(gpi)}
                            y2="30"
                            stroke="black"
                            stroke-width="1"
                        />
                        <text
                            x={gpi_scaling(gpi) - 5}
                            y="20"
                            paint-order="stroke"
                            stroke="black"
                            fill="white"
                            stroke-linecap="butt"
                            stroke-linejoin="miter"
                            text-anchor="end">{gpi}</text
                        >
                        <text
                            x="5"
                            y="40"
                            fill="white"
                            font-size="10"
                            text-anchor="start">more peaceful</text
                        >
                        <text
                            x={vis_width}
                            y="40"
                            fill="white"
                            font-size="10"
                            text-anchor="end">less peaceful</text
                        >
                    </svg>
                </div>
                <br />

                <p style="margin-bottom: 5px;">
                    Corruption Perception Index Ranking:
                </p>
                <div id="cpi">
                    <svg height="45px" width={vis_width}>
                        <defs>
                            <linearGradient id="Gradient2">
                                <stop class="stop11" offset="0%" />
                                <stop class="stop22" offset="50%" />
                                <stop class="stop33" offset="100%" />
                            </linearGradient>
                        </defs>
                        <rect
                            id="rect2"
                            x="5"
                            rx="2"
                            y="0"
                            width={vis_width - 5}
                            height="30"
                        />
                        <line
                            x1={cpi_scaling(cpi)}
                            y1="0"
                            x2={cpi_scaling(cpi)}
                            y2="30"
                            stroke="black"
                            stroke-width="1"
                        />
                        <text
                            x={cpi_scaling(cpi) - 5}
                            y="20"
                            paint-order="stroke"
                            stroke="black"
                            fill="white"
                            stroke-linecap="butt"
                            stroke-linejoin="miter"
                            text-anchor="end">{cpi}</text
                        >
                        <text
                            x="5"
                            y="40"
                            fill="white"
                            font-size="10"
                            text-anchor="start">less corrupt</text
                        >
                        <text
                            x={vis_width}
                            y="40"
                            fill="white"
                            font-size="10"
                            text-anchor="end">more corrupt</text
                        >
                    </svg>
                </div>

                <br />

                {@html selected_country_details?.general_updates}
            </div>
        </div>
        <div id="peace_process">
            <h5>Peace Process</h5>
            <div class="scrollable-content">
                {@html selected_country_details?.peace_process_text}
            </div>
        </div>
        <div id="tracker">
            <h5>Additional Information</h5>
            <div class="content-wrapper">
                <div class="content-box">
                    <h5 style="background-color: #2a2a2a">Tracker</h5>
                    <div id="tracker_link">
                        <a href={tracker_link} target="_blank"
                            ><img src="./pax.png" alt="pax logo" /></a
                        >
                    </div>
                </div>
                <div class="content-box">
                    <h5 style="background-color: #2a2a2a">Search PA-X</h5>
                    <div id="pax_link">
                        <a href={pax_link} target="_blank"
                            ><img src="./search.png" alt="search icon" /></a
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

    #general,
    #peace_process {
        background: #1a1c25;
        flex-grow: 2;
        flex-basis: 0;
        width: 100%;
        overflow-y: auto;
        display: flex;
        flex-direction: column;
    }

    #overview,
    #tracker {
        background: #1a1c25;
        flex-grow: 1;
        flex-basis: 0;
        width: 100%;
        overflow-y: auto;
        display: flex;
        flex-direction: column;
    }

    #general,
    #peace_process,
    #overview,
    #tracker {
        background: #1a1c25;
    }

    #overview h5,
    #general h5 {
        background-color: #444444; /* Ensure headers have a background */
        z-index: 2; /* Ensure they stay above other content */
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

    /* Add this CSS to the existing style */
    .row {
        display: flex;
        flex-grow: 1;
        justify-content: space-between;
        align-items: center;
        gap: 10px;
    }

    .row > div {
        flex-basis: 50%;
        text-align: center;
    }

    .tooltip-container {
        position: relative;
        display: inline-block;
    }

    .tooltip-container .tooltip-text {
        visibility: hidden;
        width: 80px;
        background-color: black;
        color: #fff;
        text-align: center;
        border-radius: 6px;
        padding: 5px;
        position: absolute;
        z-index: 10;
        bottom: 125%;
        left: 50%;
        transform: translateX(-50%);
        opacity: 0;
        transition: opacity 0.3s;
        font-size: 0.8em;
    }

    .tooltip-container .tooltip-text::after {
        content: "";
        position: absolute;
        top: 100%;
        left: 50%;
        transform: translateX(-50%);
        border-width: 5px;
        border-style: solid;
        border-color: #555 transparent transparent transparent;
    }

    .tooltip-container:hover .tooltip-text {
        visibility: visible;
        opacity: 1;
    }

    .content-box {
        flex-basis: 50%;
        background: #1a1c25;
        color: white;
        padding: 5px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.8);
        display: flex;
        flex-direction: column;
        box-sizing: border-box;
        /* overflow: hidden; Prevent overflow from disrupting layout */
        overflow: visible;
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

    #gpi,
    #cpi {
        width: 100%;
        height: 30px;
    }

    #cpi {
        margin-bottom: 10px;
    }

    .scrollable-content {
        flex-grow: 1;
        overflow-y: auto;
        padding: 10px 15px;
        background: none;
        box-sizing: border-box;
    }

    h5 {
        position: sticky; /* Make header sticky */
        top: 0; /* Stick to the top of the parent */
        z-index: 1; /* Ensure it stays on top of scrollable content */
        margin: 0;
        padding: 2px 15px;
        background-color: #444444;
        color: white;
        font-size: 1em;
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
        margin: 0px;
    }

    #rect1 {
        fill: url(#Gradient1);
    }

    .stop1 {
        stop-color: #e8d4cc;
    }
    .stop2 {
        stop-color: #f39267;
    }
    .stop3 {
        stop-color: #ff501b;
    }

    #rect2 {
        fill: url(#Gradient2);
    }

    .stop11 {
        stop-color: #dadff1;
    }
    .stop22 {
        stop-color: #6c7ec6;
    }
    .stop33 {
        stop-color: #242f5c;
    }
</style>
