<script>
  import world from "$data/110m.json";
  import * as topojson from "topojson-client";

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
    geoEckert1,
    // geoInterruptedHomolosine, // lo dejo fuera si no lo usas
  } from "d3-geo-projection";

  import { geoAirocean } from "d3-geo-polygon";

  const graticule30 = geoGraticule().step([30, 30]); // [longitud, latitud] en grados

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
    { name: "Mercator", proj: () => geoMercator() },
    { name: "Albers", proj: () => geoAlbers() },
    { name: "Dymaxion", proj: () => geoAirocean() },
    { name: "Estereográfica (polo norte)", proj: northPoleStereographic },
    { name: "Equal Earth", proj: () => geoEqualEarth() },
    { name: "Winkel Tripel", proj: () => geoWinkel3() },
    { name: "Mollweide", proj: () => geoMollweide() },
    { name: "Robinson", proj: () => geoRobinson() },
    { name: "Azimutal", proj: () => geoAzimuthalEquidistant() },
    { name: "Ortográfica", proj: () => geoOrthographic() },
    { name: "Bonne", proj: () => geoBonne() },
    { name: "Eckert 1", proj: () => geoEckert1() },
    { name: "Gall-Peters", proj: () => geoCylindricalEqualArea().parallel(45) },
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
</script>

<div class="chart-container" bind:clientWidth={width}>
  <div class="proj-list">
    {#each projs as p, i}
      <button
        class:active={i === current}
        on:click={() => selectProjection(i)}
        title={p.name}
      >
        {p.name}
      </button>
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

<style>
  .chart-container {
    display: flex;
    flex-direction: row; /* botones a la izquierda en escritorio */
    max-width: 1000px;
    margin: 0 auto;
    height: 350px;
  }

  .chart-container svg {
    width: 100%; /* SVG ocupa todo el ancho disponible */
    height: auto;
    max-height: 350px;
    margin-left: -60px; /* opcional, ajusta según necesidad */
  }

  /* Lista de botones */
  .proj-list {
    display: flex;
    flex-direction: column; /* vertical en escritorio */
    justify-content: flex-start;
    flex: 0 0 180px;
    height: 100%;
    gap: 0.32rem;
  }

  .proj-list button {
    flex: 1;
    border-radius: 6px;
    border: 0.25px solid black;
    background: transparent;
    cursor: pointer;
    text-align: left;
  }

  /* Botones activos */
  .proj-list button.active {
    background: #5ca7e6;
    color: #fff;
    font-weight: 600;
    border-color: #222;
  }

  .proj-list button:not(.active):hover {
    background: #eee;
  }

  /* MÓVIL */
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
  }
</style>
