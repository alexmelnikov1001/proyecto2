<script>
  import * as d3 from "d3"
  import futbolistas from "/src/data/athletes.csv"

  console.log("futbolistas", futbolistas)


  function getBotin(color) {
    if (color === "rojo") return "/images/botines/rojo.png";
    if (color === "naranja") return "/images/botines/naranja.png";
    if (color === "azul") return "/images/botines/azul.png";
    if (color === "verde") return "/images/botines/verde.png";
  }

  function getCordones(cordones) {
    if (cordones === "atados") return "/images/cordones/atados.png";
    if (cordones === "desatados") return "/images/cordones/desatados.png";
  }

  function getPunta(punta) {
    if (punta === "normal") return "/images/puntas/normal.png";
    if (punta === "dorada") return "/images/puntas/dorada.png";
  }

  function getSuela(suela) {
    if (suela === "nueva") return "/images/suelas/nueva.png";
    if (suela === "usada") return "/images/suelas/usada.png";
    if (suela === "gastada") return "/images/suelas/gastada.png";
  }
</script>

<div class="container">
  {#each futbolistas as f}
    <div class="jugador">
      <div class="botin-contenedor">
        <img class="capa" src={getBotin(f.botin)} alt="botin" />
        <img class="capa" src={getSuela(f.suela)} alt="suela" />
        <img class="capa" src={getCordones(f.cordones)} alt="cordones" />
        <img class="capa" src={getPunta(f.punta)} alt="punta" />
      </div>
      <p class="nombre">{f.nombre}</p>
    </div>
  {/each}
</div>

<style>
  .container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 30px;
    margin-top: 40px;
  }

  .jugador {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .botin-contenedor {
    position: relative;
    width: 150px;
    height: 150px;
  }

  .capa {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    object-fit: contain;
  }

  .nombre {
    margin-top: 8px;
    font-size: 14px;
    font-weight: bold;
    text-align: center;
  }
</style>











<!-- 

  /* 1. Escala para participaciones (cuantitativo > grosor) */
  const minMaxParticipations = d3.extent(atletas, (d) => d.participations)
  let grosorPartic = d3.scaleLinear()
    .domain(minMaxParticipations).range([2, 18])

  /* 2. Escala para medallas (cuantitativo > diámetro círculo) *///
  const maxMedallas = d3.max(atletas, (d) => d.medallas)
  const diamMedallas = d3.scaleRadial()
    .domain([0, maxMedallas]).range([0, 100])

  /* 3. Escala para genero (categórico > color) */
  const colorGenero = d3.scaleOrdinal()
    .domain(["F", "M"])
    .range(["#F7DDBA", "#E4D9F2"])

  /* 4. Escala para continentes (categórico > color)   */
  const colorContinentes = d3
    .scaleOrdinal()
    .domain(["América", "África", "Asia", "Europa", "Oceanía"])
    .range(["#ed334e", "#000000", "#fbb132", "#009fe3", "#00963f"])

</script>

<main>
  <div class="header">
    <img src="/images/logo_referencias.svg" width="190" alt="anillos" />
    <h3 class="headline">
      <b>Los reyes del oro</b>
      Medallas, participaciones y dominio en distintos continentes
    </h3>
    <p class="bajada">
      Los atletas con más medallas olímpicas de oro en los Juegos Olímpicos
    </p>
    <img
      class="referencias"
      src="/images/referencias.svg"
      width="490"
      alt="anillos"
    />
  </div>

   Conedor de las entidades
    <div class="container">
      
      Iteramos la data para visualizar c/ entidad
      {#each atletas as atleta}
        <div class="person-container">
          <div
            class="person"
            style="
              border-color: {colorContinentes(atleta.continent)};
              background-color:{colorGenero(atleta.gender)}; 
              width: {diamMedallas(atleta.medallas)}px; 
              height: {diamMedallas(atleta.medallas)}px; 
              border-width: {grosorPartic(atleta.participations)}px; 
            ">
          </div>
        </div>
      {/each}

    </div>
</main>

<style>
  .header {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    margin-top: 20px;
    margin-bottom: 80px;
  }
  .headline {
    font-size: 40px;
    font-weight: 300;
    line-height: 1.2;
    text-align: center;
    margin: 20px;
  }
  .bajada {
    font-size: 24px;
    font-weight: 300;
    text-align: center;
    margin: 10px;
  }
  .headline b {
    display: block;
  }
  .container {
    display: flex;
    justify-content: center;
    align-items: end;
    margin: auto;
    flex-wrap: wrap;
    max-width: 1020px;
    gap: 30px;
    margin-bottom: 100px;
  }
  .person-container {
    display: flex;
    justify-content: center;
    flex-direction: column;
    align-items: center;
    flex: 180px 0 0;
  }
  .person {
    width: 100px;
    height: 100px;
    border: 10px solid black;
    border-radius: 50%;
    box-sizing: border-box;
    background-color: pink;
  }
  .nombre {
    font-size: 13px;
    font-weight: bold;
    line-height: 1;
    text-transform: uppercase;
    margin: 0;
    margin-top: 8px;
  }
  .deporte {
    font-size: 14px;
    color: #666;
    margin: 0;
  }
  .referencias {
    margin-top: 50px;
    margin-bottom: 20px;
  }
</style>

-->