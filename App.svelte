<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap" rel="stylesheet">


<script>
  import * as d3 from "d3"
  import futbolistas from "/src/data/futbolistas.csv"

  console.log("futbolistas", futbolistas)

  function getBotin(continente) {
    if (continente === "Europa") return "/images/azul.png";
    if (continente === "América") return "/images/verde.png";
    if (continente === "África") return "/images/rojo.png";
    if (continente === "Asia") return "/images/naranja.png";
  }

  function getCordones(estado, gano_mundial) {
    if (estado === "A")
      if (gano_mundial == "T") return "/images/atado_dorado.png";
      else return "/images/atado_normal.png";

    else
      if (gano_mundial == "T") return "/images/desatado_dorado.png";
      else return "/images/desatado_normal.png";
  }

  function getSuela(tiempo_en_primera) {
    const anios = parseInt(tiempo_en_primera);
    if (anios > 17) return "/images/suela_gastada.png";
    if (anios >= 10) return "/images/suela_usada.png";
    return "/images/suela_nueva.png";
  }

</script>

<h1 class="titulo">FUTBOL CODIFICADO EN BOTINES</h1>

<div class="leyenda">
  <div class="leyenda-color">
    <h2 class="leyenda-titulo">COLOR</h2>
    <div class="fila-items">
      <div class="item">
        <img src="/images/pintura_verde.png" alt="verde" />
        <span>América</span>
      </div>
      <div class="item">
        <img src="/images/pintura_naranja.png" alt="naranja" />
        <span>Asia</span>
      </div>
      <div class="item">
        <img src="/images/pintura_roja.png" alt="rojo" />
        <span>África</span>
      </div>
      <div class="item">
        <img src="/images/pintura_azul.png" alt="azul" />
        <span>Europa</span>
      </div>
    </div>

    <h2 class="leyenda-titulo">SUELA</h2>
    <div class="fila-items">
      <div class="item">
        <img src="/images/tapon_nuevo.png" alt="nuevo" />
        <span>Menos de 10 años</span>
      </div>
      <div class="item">
        <img src="/images/tapon_usado.png" alt="usado" />
        <span>10 a 17 años</span>
      </div>
      <div class="item">
        <img src="/images/tapon_gastado.png" alt="gastado" />
        <span>Más de 17 años</span>
      </div>
    </div>

    <h2 class="leyenda-titulo">CORDONES</h2>
    <div class="fila-items">
      <div class="item">
        <img src="/images/atado_normal.png" alt="atado normal" />
        <span>Atado (sin mundial)</span>
      </div>
      <div class="item">
        <img src="/images/atado_dorado.png" alt="atado dorado" />
        <span>Atado (ganó mundial)</span>
      </div>
      <div class="item">
        <img src="/images/desatado_normal.png" alt="desatado normal" />
        <span>Desatado (sin mundial)</span>
      </div>
      <div class="item">
        <img src="/images/desatado_dorado.png" alt="desatado dorado" />
        <span>Desatado (ganó mundial)</span>
      </div>
    </div>
  </div>
</div>



<div class="container">
  {#each futbolistas as f}
    <div class="jugador">
      <div class="botin-contenedor">
        <img class="capa" src={getBotin(f.continente)} alt="botin" />
        <img class="capa" src={getSuela(f.tiempo_en_primera)} alt="suela" />
        <img class="capa" src={getCordones(f.estado, f.gano_mundial)} alt="cordones+punta" />
      </div>
      <p class="nombre">{f.nombre}</p>
    </div>
  {/each}
</div>

<style>

.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 4 columnas fijas */
  gap: 30px;
  justify-items: center;
  margin-top: 30px;
}

.jugador {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.botin-contenedor {
  position: relative;
  width: 380px;   /* antes 320px */
  height: 380px;  /* antes 320px */
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
  margin-top: -100px;
  font-size: 22px;
  font-weight: 600;
  text-align: center;
}

.titulo {
  font-size: 45px;
  font-weight: 700;
  text-align: center;
  margin-top: 30px;
  margin-bottom: 10px;
  font-family: 'Poppins';

}

.leyenda {
  width: 90%;
  max-width: 15000px;
  height: 700px;
  margin: 20px auto 40px;
  background-color: #f5f5f5;
  border: 2px solid #ccc;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
  margin-top: 40px;
}


.leyenda-color {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 40px;
}

.leyenda-titulo {
  font-size: 28px;
  font-weight: 700;
  margin: 30px 0 20px;
  font-family: 'Poppins', sans-serif;
}

.fila-items {
  display: flex;
  flex-wrap: wrap;
  gap: 50px;
  justify-content: center;
  margin-bottom: 20px;
}

.item {
  display: flex;
  align-items: center;
  gap: 12px;
  min-width: 220px;
}

.item img {
  width: 100px;
  height: auto;
}

.item span {
  font-size: 18px;
  font-weight: 600;
  font-family: 'Poppins', sans-serif;
}


</style>