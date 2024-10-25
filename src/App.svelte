<script>
  import * as topojson from "topojson-client";
  import estadosPreferencia from "$data/estadosPreferencia.js";
  import distritosPreferencia from "$data/distritosPreferencia.js";
  import icon from "./data/refresh.svg";
  import icon1 from "./data/info.svg";
  import kamala from "./data/kamala.svg";
  import trump from "./data/trump.svg";
  import {
    json,
    geoAlbersUsa,
    geoPath,
    scaleBand,
    scaleLinear,
    select,
  } from "d3";

  let width;
  $: height = width / 1.7;

  const topojsonPath =
    "https://cdn.jsdelivr.net/npm/us-atlas@3/states-10m.json";
  let geojson;

  json(topojsonPath).then((topology) => {
    geojson = topojson.feature(topology, topology.objects.states);
  });

  $: projection = geoAlbersUsa().fitSize([width, height - 10], geojson);
  $: pathGenerator = geoPath(projection);

  let initialStates = [];
  let states = [];
  $: if (geojson) {
    initialStates = geojson.features.map((feature) => {
      const stateData = estadosPreferencia.find(
        (state) => state.id === feature.id,
      );
      const abbreviation = stateData ? stateData.abbreviation : "N/A";
      const votePreference = stateData ? stateData.vote_preference : "N/A";
      const votes = stateData ? stateData.votes : "N/A";
      const centroid = pathGenerator.centroid(feature);

      return {
        ...feature,
        path: pathGenerator(feature),
        centroid: centroid,
        abbreviation: stateData ? stateData.abbreviation : "N/A",
        votePreference: stateData ? stateData.vote_preference : "N/A",
        votes: stateData ? stateData.votes : "N/A",
      };
    });

    states = [...initialStates];
  }

  let originalBallColors = {
    nebraska: ["#E6322E", "#1597D9", "#E6322E"],
    maine: ["#1597D9", "#E6322E"],
    dc: ["#1597D9"],
  };

  function resetStates() {
    states = [...initialStates];

    const infoBlocks = document.querySelectorAll(".info-block");
    infoBlocks.forEach((block, index) => {
      const balls = block.querySelectorAll(
        ".bola, .bola-grande, .bola-extra-grande",
      );
      balls.forEach((ball, ballIndex) => {
        ball.style.backgroundColor = originalBallColors[block.id][ballIndex];
      });
    });
  }

  let hoveredStateId = null;

  function toggleVotePreference(stateId) {
    const stateData = estadosPreferencia.find((state) => state.id === stateId);

    if (stateData) {
      stateData.vote_preference =
        stateData.vote_preference === "Republican"
          ? "Swing"
          : stateData.vote_preference === "Democrat"
            ? "Republican"
            : stateData.vote_preference === "Swing"
              ? "Democrat"
              : "Democrat";

      states = states.map((state) =>
        state.id === stateId
          ? { ...state, votePreference: stateData.vote_preference }
          : state,
      );
    }
  }

  function recalcularVotos() {
  votesDemocrats = states.reduce((total, state) =>
    state.votePreference === "Democrat" ? total + (state.votes || 0) : total, 0
  );

  votesRepublicans = states.reduce((total, state) =>
    state.votePreference === "Republican" ? total + (state.votes || 0) : total, 0
  );

  votesSwing = states.reduce((total, state) =>
    state.votePreference === "Swing" ? total + (state.votes || 0) : total, 0
  );

  votesDemocrats += distritosPreferencia.reduce((total, district) =>
    district.vote_preference === "Democrat" ? total + (district.votes || 0) : total, 0
  );

  votesRepublicans += distritosPreferencia.reduce((total, district) =>
    district.vote_preference === "Republican" ? total + (district.votes || 0) : total, 0
  );

  votesSwing += distritosPreferencia.reduce((total, district) =>
    district.vote_preference === "Swing" ? total + (district.votes || 0) : total, 0
  );
}
function toggleDistrictVotePreference(districtNumber) {
  const districtData = distritosPreferencia.find(
    (district) => district.number === districtNumber
  );

  if (districtData) {
    districtData.vote_preference =
      districtData.vote_preference === "Republican"
        ? "Swing"
        : districtData.vote_preference === "Democrat"
          ? "Republican"
          : districtData.vote_preference === "Swing"
            ? "Democrat"
            : "Democrat";
  }

  $: recalcularVotos();
}

  let tooltipText = "";
  let tooltipVisible = false;
  let tooltipPosition = { top: 0, left: 0 };

  function showTooltip(event, text) {
    tooltipText = text;
    tooltipVisible = true;
    const icon = event.currentTarget;
    const iconRect = icon.getBoundingClientRect();

    tooltipPosition = {
      top: iconRect.top,
      left: iconRect.left,
    };
  }

  function hideTooltip() {
    tooltipVisible = false;
  }

  let votesDemocrats = 0;
  let votesRepublicans = 0;
  let votesSwing = 0;

  $: {
  votesDemocrats = states.reduce((total, state) => 
    state.votePreference === "Democrat" ? total + (state.votes || 0) : total, 0
  );

  votesRepublicans = states.reduce((total, state) => 
    state.votePreference === "Republican" ? total + (state.votes || 0) : total, 0
  );

  votesSwing = states.reduce((total, state) => 
    state.votePreference === "Swing" ? total + (state.votes || 0) : total, 0
  );

  votesDemocrats += distritosPreferencia.reduce((total, district) => 
    district.vote_preference === "Democrat" ? total + (district.votes || 0) : total, 0
  );

  votesRepublicans += distritosPreferencia.reduce((total, district) => 
    district.vote_preference === "Republican" ? total + (district.votes || 0) : total, 0
  );

  votesSwing += distritosPreferencia.reduce((total, district) => 
    district.vote_preference === "Swing" ? total + (district.votes || 0) : total, 0
  );
}

  window.addEventListener("DOMContentLoaded", (event) => {
    function updateIframeHeight() {
      const el = document.documentElement;
      const rect = el.getBoundingClientRect();
      const styles = window.getComputedStyle(el);
      const margin =
        parseFloat(styles.marginTop) + parseFloat(styles.marginBottom);
      const height = Math.ceil(rect.height + margin);

      window.parent.postMessage(
        {
          type: "resize-iframe",
          value: height,
        },
        "*",
      );
    }
    updateIframeHeight();

    if (window.ResizeObserver) {
      new ResizeObserver(() => {
        updateIframeHeight();
      }).observe(document.documentElement);
    } else {
      window.addEventListener("load", updateIframeHeight);
      window.addEventListener("resize", updateIframeHeight);
    }

    window.addEventListener(
      "message",
      (event) => {
        if (event.data.type === "request-resize") {
          updateIframeHeight();
        }
      },
      false,
    );
  });

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<main bind:clientWidth={width}>
<div style="display: flex; justify-content: center; margin-top: 20px; margin-bottom: 10px;">
  <img
  src={kamala}
  alt="Kamala Harris"
  style="height: {width > 500 ? "50px" : "40px"}; position: relative; top: {width > 500 ? "-18px" : "-14px"}; right: -2px;"
  class="candidate-icon" />  
  <svg width={width * 0.8} height="40">
        <rect 
            width={votesDemocrats / 538 * (width * 0.8)} 
            height="12" 
            fill="#1597D9" 
        />
        
        <rect 
            width={votesSwing / 538 * (width * 0.8)} 
            height="12" 
            fill="#B3B3B3" 
            x={votesDemocrats / 538 * (width * 0.8)} 
        />
        
        <rect 
            width={votesRepublicans / 538 * (width * 0.8)} 
            height="12" 
            fill="#E6322E" 
            x={(votesDemocrats + votesSwing) / 538 * (width * 0.8)} 
        />

        <polygon 
            points={`${(270 / 538) * (width * 0.8)}, 15 
                     ${(270 / 538) * (width * 0.8) - 5}, 20 
                     ${(270 / 538) * (width * 0.8) + 5}, 20`}
            fill="black" 
        />

        <text 
        x={(270 / 538) * (width * 0.8)} 
        y="35" 
        text-anchor="middle" 
        fill="black" 
        font-size="0.8em"
        font-style="italic">
        {width > 500 ? "Mayoría: 270" : "270"}
    </text>
        
        <text 
            x="5" 
            y="28" 
            text-anchor="start" 
            fill="#1597D9"
            font-weight="800"
            font-size={votesDemocrats > 269 ? "1em" : "0.85em"}>
            Kamala Harris {votesDemocrats}
        </text>
        
        <text 
            x={(votesDemocrats + votesSwing + votesRepublicans) / 538 * (width * 0.8) - 5} 
            y="28" 
            text-anchor="end" 
            font-weight="800"
            fill="#E6322E" 
            font-size={votesRepublicans > 269 ? "1em" : "0.85em"}>
            {votesRepublicans} Donald Trump 
        </text>
    </svg>
    <img src={trump} class="candidate-icon" alt="Donald Trump" style="height: {width > 500 ? "50px" : "40px"}; position: relative; top: {width > 500 ? "-18px" : "-14px"}; left: -3px" />
</div>
  <button on:click={resetStates}>
    <img src={icon} alt="Icono" class="reset-icon" />
    Restablecer
  </button>
  <svg {width} {height}>
    {#each states as state}
      {#if state.path}
        <path
          d={state.path}
          class:active={hoveredStateId === state.id}
          style="cursor: pointer;"
          on:mouseenter={() => (hoveredStateId = state.id)}
          on:mouseleave={() => (hoveredStateId = null)}
          on:click={() => {
            toggleVotePreference(state.id);
          }}
          fill={state.votePreference === "Republican"
            ? "#E6322E"
            : state.votePreference === "Democrat"
              ? "#1597D9"
              : "#B3B3B3"}
          stroke="white"
          stroke-width="0.5"
        />
        {#if state.abbreviation !== "MA" && state.abbreviation !== "RI" && state.abbreviation !== "CT" && state.abbreviation !== "NJ" && state.abbreviation !== "DE" && state.abbreviation !== "MD" && state.abbreviation !== "DC"}
          <text
            class="abreviaciones"
            x={state.abbreviation === "Calif." || state.abbreviation === "Luis."
              ? state.centroid[0] - 10
              : state.abbreviation === "Flo."
                ? state.centroid[0] + 12
                : state.abbreviation === "Hawái"
                  ? state.centroid[0] - 35
                  : state.abbreviation === "Mích."
                    ? state.centroid[0] + 5
                    : state.centroid[0]}
            y={state.abbreviation === "Alaska"
              ? state.centroid[1] - 10
              : state.abbreviation === "Idaho"
                ? state.centroid[1] + 8
                : state.abbreviation === "Mích."
                  ? state.centroid[1] + 12
                  : state.centroid[1]}
            text-anchor="middle"
            font-size="10px"
            font-weight="600"
            fill={state.abbreviation === "Hawái" ? "black" : "white"}
          >
            {state.abbreviation}
          </text>
          <text
            class="abreviaciones numero"
            x={state.abbreviation === "Calif." || state.abbreviation === "Luis."
              ? state.centroid[0] - 10
              : state.abbreviation === "Flo."
                ? state.centroid[0] + 12
                : state.abbreviation === "Hawái"
                  ? state.centroid[0] - 35
                  : state.abbreviation === "Mích."
                    ? state.centroid[0] + 5
                    : state.centroid[0]}
            y={state.abbreviation === "Alaska"
              ? state.centroid[1]
              : state.abbreviation === "Idaho"
                ? state.centroid[1] + 18
                : state.abbreviation === "Mích."
                  ? state.centroid[1] + 22
                  : state.centroid[1] + 10}
            text-anchor="middle"
            font-size="10px"
            fill={state.abbreviation === "Hawái" ? "black" : "white"}
          >
            {state.votes}
          </text>
        {/if}
      {/if}
    {/each}

    {#each states as state}
      {#if state.path && hoveredStateId === state.id}
        <path
          d={state.path}
          class="active"
          fill="none"
          stroke="black"
          stroke-width="1"
          user-select="none"
          pointer-events="none"
        />
      {/if}
    {/each}
  </svg>
  <div class="info-blocks">
    <div class="info-block" id="nebraska">
      <div class="header-container">
        <img
          src={icon1}
          alt="Icono"
          class="info-icon"
          on:mouseenter={(event) =>
            showTooltip(
              event,
              "Nebraska elige dos compromisarios a nivel estatal y uno por cada distrito.",
            )}
          on:mouseleave={hideTooltip}
        />
        <h2>Nebraska</h2>
      </div>
      <div class="bolitas">
        <span
          class="bola"
          style="background-color: #E6322E;"
          on:click={(e) => {
            e.currentTarget.style.backgroundColor =
              e.currentTarget.style.backgroundColor === "rgb(179, 179, 179)"
                ? "rgb(20, 142, 204)"
                : e.currentTarget.style.backgroundColor === "rgb(20, 142, 204)"
                  ? "#E6322E"
                  : "#B3B3B3";

            toggleDistrictVotePreference("N02");
          }}>1</span
        >
        <span
          class="bola"
          style="background-color: #148ecc;"
          on:click={(e) => {
            e.currentTarget.style.backgroundColor =
              e.currentTarget.style.backgroundColor === "rgb(179, 179, 179)"
                ? "rgb(20, 142, 204)"
                : e.currentTarget.style.backgroundColor === "rgb(20, 142, 204)"
                  ? "#E6322E"
                  : "#B3B3B3";

            toggleDistrictVotePreference("N03");
          }}>1</span
        >
        <span
          class="bola"
          style="background-color: #E6322E;"
          on:click={(e) => {
            e.currentTarget.style.backgroundColor =
              e.currentTarget.style.backgroundColor === "rgb(179, 179, 179)"
                ? "rgb(20, 142, 204)"
                : e.currentTarget.style.backgroundColor === "rgb(20, 142, 204)"
                  ? "#E6322E"
                  : "#B3B3B3";

          toggleDistrictVotePreference("N04");
          }}>1</span
        >
      </div>
    </div>
    <div class="info-block" id="maine">
      <div class="header-container">
        <img
          src={icon1}
          alt="Icono"
          class="info-icon"
          on:mouseenter={(event) =>
            showTooltip(
              event,
              "Maine elige dos compromisarios a nivel estatal y uno por cada distrito.",
            )}
          on:mouseleave={hideTooltip}
        />
        <h2>Maine</h2>
      </div>
      <div class="bolitas">
        <span
          class="bola"
          style="background-color: #148ecc;"
          on:click={(e) => {
            e.currentTarget.style.backgroundColor =
              e.currentTarget.style.backgroundColor === "rgb(179, 179, 179)"
                ? "rgb(20, 142, 204)"
                : e.currentTarget.style.backgroundColor === "rgb(20, 142, 204)"
                  ? "#E6322E"
                  : "#B3B3B3";

            toggleDistrictVotePreference("M02");
          }}>1</span
        >
        <span
          class="bola"
          style="background-color: #E6322E;"
          on:click={(e) => {
            e.currentTarget.style.backgroundColor =
              e.currentTarget.style.backgroundColor === "rgb(179, 179, 179)"
                ? "rgb(20, 142, 204)"
                : e.currentTarget.style.backgroundColor === "rgb(20, 142, 204)"
                  ? "#E6322E"
                  : "#B3B3B3";

            toggleDistrictVotePreference("M03");
          }}>1</span
        >
      </div>
    </div>
    <div class="info-block" id="dc">
      <div class="header-container">
        <img
          src={icon1}
          alt="Icono"
          class="info-icon"
          on:mouseenter={(event) =>
            showTooltip(
              event,
              "Aunque no es un estado, Washington D.C. elige tres compromisarios.",
            )}
          on:mouseleave={hideTooltip}
        />
        <h2>Washington D.C.</h2>
      </div>
      <div class="bolitas">
        <span
          class="bola-extra-grande"
          style="background-color: #148ecc;"
          on:click={(e) => {
            e.currentTarget.style.backgroundColor =
              e.currentTarget.style.backgroundColor === "rgb(179, 179, 179)"
                ? "rgb(20, 142, 204)"
                : e.currentTarget.style.backgroundColor === "rgb(20, 142, 204)"
                  ? "rgb(230, 50, 46)"
                  : "#B3B3B3";
                  
            toggleDistrictVotePreference("W01");
          }}>3</span
        >
      </div>
    </div>
  </div>
  {#if tooltipVisible}
    <div
      class="tooltip"
      style="top: {tooltipPosition.top}px; left: {tooltipPosition.left}px"
    >
      {tooltipText}
    </div>
  {/if}
</main>

<style>
  main {
    max-width: 1500px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .abreviaciones {
    user-select: none;
    pointer-events: none;
    font-size: 8px;
  }

  @media (max-width: 699px) {
    .abreviaciones {
      display: none;
    }
  }

  @media (min-width: 900px) {
    .abreviaciones {
      font-size: 11px;
    }
  }

  @media (min-width: 900px) {
    .numero {
      transform: translateY(2px);
    }
  }

  @media (min-width: 1000px) {
    .abreviaciones {
      font-size: 12px;
    }
  }

  button {
    padding: 5px;
    border: 1px solid grey;
    border-radius: 10px;
    background-color: white;
    color: black;
    cursor: pointer;
    transition: background-color 0.3s ease;
    font-size: 0.8em;
    display: flex;
    align-items: center;
    
  }

  button:hover {
    background-color: #cccccc;
    border: 1px solid black;
  }

  .info-blocks {
    display: flex;
    margin-top: 5px;
  }

  .info-block {
    padding: 15px;
    width: 33%;
    text-align: left;
  }

  .info-block h2 {
    font-size: 0.8em;
    font-weight: 600;
    white-space: nowrap;
  }

  .tooltip {
    position: absolute;
    background-color: white;
    color: black;
    padding: 10px;
    border-radius: 5px;
    font-size: 0.8em;
    font-weight: 500;
    max-width: 150px;
    word-wrap: break-word;
    z-index: 1000;
    pointer-events: none;
    transform: translateY(-100%);
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
  }

  .reset-icon {
    width: 12px;
    height: 13px;
    margin-right: 3px;
  }

  .info-icon {
    width: 14px;
    height: 14px;
    margin-right: 3px;
  }

  .bolitas {
    display: flex;
    gap: 4px;
    text-align: center;
    font-size: 10px;
    align-items: end;
    color: white;
    text-anchor: start;
  }

  .bola,
  .bola-extra-grande {
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 10%;
    border: 1px solid grey;
    color: white;
    cursor: pointer;
    user-select: none;
  }

  .bola {
    width: 12px;
    height: 12px;
  }

  .bola-extra-grande {
    width: 36px;
    height: 12px;
  }

  .bola:hover,
  .bola-extra-grande:hover {
    border: 1px solid black;
  }

  .header-container {
    display: flex;
    align-items: center;
    margin: 0 0 5px;
  }

</style>
