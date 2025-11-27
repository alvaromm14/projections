<script>
  import world from "$data/110m.json";
  import * as topojson from "topojson-client";
  import Tooltip from "$components/Tooltip.svelte";
  import { tick } from "svelte";

  import {
    geoPath,
    geoGraticule,
    geoMercator,
    geoEqualEarth,
    geoOrthographic,
    geoAzimuthalEquidistant,
    geoStereographic,
    geoAlbers,
  } from "d3-geo";
  import {
    geoWinkel3,
    geoCylindricalEqualArea,
    geoMollweide,
    geoRobinson,
    geoBonne,
    geoEckert4,
  } from "d3-geo-projection";

  import { geoAirocean } from "d3-geo-polygon";

  const graticule30 = geoGraticule().step([30, 30]);

  let countries = topojson.feature(world, world.objects.countries).features;
  let borders = topojson.mesh(
    world,
    world.objects.countries,
    (a, b) => a !== b,
  );

  let width;
  let height = 350;

  const northPoleStereographic = () =>
    geoStereographic()
      .center([0, 90])
      .rotate([0, -90])
      .clipAngle(90)
      .precision(0.1);

  const projs = [
    {
      name: "Mercator",
      proj: () => geoMercator(),
      description:
        "Proyección conforme que conserva ángulos y direcciones.Muy usada en navegación, pero distorsiona áreas cerca de los polos.",
    },
    {
      name: "Equal Earth",
      proj: () => geoEqualEarth(),
      description:
        "Proyección pseudocilíndrica equivalente que conserva el tamaño relativo de las áreas.Pretende combinar exactitud y estética.",
    },
    {
      name: "Gall-Peters",
      proj: () => geoCylindricalEqualArea().parallel(45),
      description:
        "Proyección cilíndrica equivalente surgida para destronar a Mercator. Representa las formas con mayor fidelidad pero estira las formas verticalmente.",
    },
    {
      name: "Albers",
      proj: () => geoAlbers(),
      description:
        "Proyección cónica equivalente con dos paralelos estándar. Útil para representar regiones con una extensión predominante de este a oeste, como Rusia.",
    },
    {
      name: "Robinson",
      proj: () => geoRobinson(),
      description:
        "Proyección convencional que representa con exactitud formas y áreas, combinando las ventajas de Mercator y Gall-Peters.",
    },
    {
      name: "Dymaxion",
      proj: () => geoAirocean(),
      description:
        "Proyección convencional basada en un icosaedro. Dibuja el territorio sin apenas distorsión como masas de tierra casi contiguas rodeadas por un único océano.",
    },
    {
      name: "Estereográfica",
      proj: northPoleStereographic,
      description:
        "Proyección conforme azimutal, centrada aquí en el polo norte.",
    },
    {
      name: "Winkel Tripel",
      proj: () => geoWinkel3(),
      description:
        "Proyección convencional que equilibra formas, distancias y direcciones. Es la utilizada por National Geographic.",
    },
    {
      name: "Mollweide",
      proj: () => geoMollweide(),
      description:
        "Proyección equivalente pseudocilíndric en la que el ecuador se representa como una línea horizontal recta perpendicular a un meridiano central.",
    },
    {
      name: "Azimutal",
      proj: () => geoAzimuthalEquidistant(),
      description:
        "Proyección acimutal o cenital que proyecta la Tierra sobre un plano tangente a ella. Conserva distancias radiales, por lo que es útil en aviación y telecomunicaciones.",
    },
    {
      name: "Ortográfica",
      proj: () => geoOrthographic(),
      description:
        "Proyección que simula la vista de la Tierra desde el espacio, generando la ilusión de estar ante un globo tridimensional.",
    },
    {
      name: "Bonne",
      proj: () => geoBonne(),
      description:
        "Proyección pseudocónica equivalente con los paralelos en forma de arcos concéntricos.",
    },
    {
      name: "Eckert IV",
      proj: () => geoEckert4(),
      description:
        "Proyección pseudocilíndrica equivalente en la que el meridiano central y los paralelos son líneas rectas.",
    },
  ];

  // índice de la proyección activa
  let current = 0;

  // proyección y path usados por el SVG
  let projection = projs[current].proj();
  let path = geoPath(projection);

  // Actualiza la proyección usando el factory de la entrada actual.
  const updateProjection = () => {
    // Obtener la fábrica y crear la proyección
    const factory = projs[current].proj;
    // algunas entradas ya devuelven la proyección al ser llamadas
    let proj = factory();

    // Ajustes comunes: escala y centrado (fitExtent para esfera)
    // Si quieres más control por proyección, añade condicionales aquí.
    if (width && typeof proj.fitExtent === "function") {
      proj = proj
        .scale(width / 2)
        .translate([width / 2, height / 2])
        .fitExtent(
          [
            [0.5, 0.5],
            [width - 0.5, height - 0.5],
          ],
          { type: "Sphere" },
        );
    } else if (width) {
      proj = proj.scale(width / 2).translate([width / 2, height / 2]);
    }

    projection = proj;
    path = geoPath(projection);
  };

  // botones de navegación
  const selectProjection = (i) => {
    current = i;
  };

  // reactivo: recalcula cuando cambian width o la proyección seleccionada
  $: if (width !== undefined) updateProjection();
  $: if (current !== undefined) updateProjection();

  let tooltipVisible = false;
  let tooltipText = "";
  let tooltipX = 0;
  let tooltipY = 0;
  let tooltipLeft = false; // para móvil

  const showTooltip = async (i, event) => {
    tooltipText = projs[i].description;
    tooltipVisible = true;

    await tick(); // espera a que el tooltip exista en el DOM

    const rect = event.target.getBoundingClientRect();
    const tooltipEl = document.querySelector(".tooltip");
    if (!tooltipEl) return;

    const tooltipRect = tooltipEl.getBoundingClientRect();
    const isMobile = window.innerWidth <= 768;

    if (isMobile) {
      // móvil: izquierda/derecha según posición del botón
      tooltipLeft = rect.left > window.innerWidth / 2;
      tooltipX = tooltipLeft ? rect.left - 8 : rect.right + 8;
      tooltipY = rect.top + 8;
    } else {
      // desktop: mantén tu lógica vertical específica
      tooltipX = rect.right + 8;
      tooltipY =
        projs[i].name === "Bonne"
          ? rect.top - 75
          : projs[i].name === "Ortográfica"
            ? rect.top - 88
            : projs[i].name === "Eckert IV"
              ? rect.top - 72
              : projs[i].name === "Azimutal"
                ? rect.top - 120
                : rect.top + rect.height / 2; // despliega hacia abajo
    }
  };

  const hideTooltip = () => {
    tooltipVisible = false;
  };
</script>

<div class="chart-container" bind:clientWidth={width}>
  <div class="proj-list">
    {#each projs as p, i}
      <div class="proj-item">
        <button
          class:active={i === current}
          on:click={() => selectProjection(i)}
        >
          {p.name}
          {#if i === current}
            <span
              class="info-icon-mobile"
              on:mouseenter={(e) => showTooltip(i, e)}
              on:mouseleave={hideTooltip}
            >
              ⓘ
            </span>
          {/if}
        </button>

        <span
          class="info-icon"
          on:mouseenter={(e) => showTooltip(i, e)}
          on:mouseleave={hideTooltip}
        >
          ⓘ
        </span>
      </div>
    {/each}
  </div>

  <svg {width} {height}>
    {#each countries as country}
      <path d={path(country)} fill="#5ca7e6" stroke="none" />
    {/each}
    <path
      d={path({ type: "Sphere" })}
      fill="none"
      stroke="black"
      stroke-width="0.75"
    />
    <path
      d={path(graticule30())}
      fill="none"
      stroke="black"
      stroke-width="0.25"
    />
  </svg>
</div>
<Tooltip
  text={tooltipText}
  visible={tooltipVisible}
  x={tooltipX}
  y={tooltipY}
  left={tooltipLeft}
/>

<style>
  .chart-container {
    display: flex;
    flex-direction: row;
    max-width: 1000px;
    margin: 0 auto;
    height: 350px;
  }

  .chart-container svg {
    width: 100%;
    height: auto;
    max-height: 350px;
    margin-left: -60px;
    pointer-events: none;
  }

  .proj-list {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    flex: 0 0 180px;
    height: 100%;
    gap: 0.32rem;
  }

  .proj-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.4rem;
    min-height: 0;
  }

  .proj-list button {
    flex: 1;
    border-radius: 6px;
    border: 0.25px solid black;
    background: transparent;
    cursor: pointer;
    text-align: left;
    padding: 0.15rem 0.4rem;
  }

  .proj-list button.active {
    background: #5ca7e6;
    color: white;
    font-weight: 600;
    border-color: #222;
  }

  .info-icon {
    font-size: 1rem;
    user-select: none;
  }

  .info-icon-mobile {
    display: none;
    margin-left: 0.25rem;
    font-size: 0.8rem;
    user-select: none;
  }

  .info-icon:hover {
    color: #5ca7e6;
  }
  .proj-list button.active {
    background: #5ca7e6;
    color: #fff;
    font-weight: 600;
    border-color: #222;
  }

  .proj-list button:not(.active):hover {
    background: #eee;
  }

  @media (max-width: 768px) {
    .chart-container {
      flex-direction: column;
      height: auto;
    }

    .proj-list {
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      justify-content: center;
      gap: 0.25rem;
      margin-bottom: 0.5rem;
      height: auto;
      flex: unset;
    }

    .proj-list button {
      flex: auto 1;
      margin-bottom: 0.25rem;
      text-align: center;
    }

    .chart-container svg {
      margin-left: 0;
    }

    .info-icon {
      display: none;
    }

    .proj-list button.active .info-icon-mobile {
      display: inline;
    }

    .proj-list button {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 0.25rem;
    }
  }
</style>
