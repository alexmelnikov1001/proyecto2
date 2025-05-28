<script>
  import * as d3 from "d3";
  import futbolistas from "/src/data/futbolistas.csv";
  import { onMount } from 'svelte';

  let mostrarLeyenda = false;
  let index = 0;

  function getBotin(continente) {
    if (continente === "Europa") return "/images/azul.png";
    if (continente === "América") return "/images/verde.png";
    if (continente === "África") return "/images/rojo.png";
    if (continente === "Asia") return "/images/naranja.png";
  }

  function getCordones(estado, gano_mundial) {
    if (estado === "A")
      return gano_mundial == "T" ? "/images/atado_dorado.png" : "/images/atado_normal.png";
    else
      return gano_mundial == "T" ? "/images/desatado_dorado.png" : "/images/desatado_normal.png";
  }

  function getSuela(tiempo_en_primera) {
    const anios = parseInt(tiempo_en_primera);
    if (anios > 17) return "/images/suela_gastada.png";
    if (anios >= 10) return "/images/suela_usada.png";
    return "/images/suela_nueva.png";
  }

  function scrollToElementoLento(id) {
    const target = document.getElementById(id);
    if (!target) return;

    const headerOffset = 90;
    const elementPosition = target.getBoundingClientRect().top;
    const offsetPosition = elementPosition + window.pageYOffset - headerOffset;

    const start = window.pageYOffset;
    const distance = offsetPosition - start;
    const duration = 500;

    let startTime = null;

    function animation(currentTime) {
      if (!startTime) startTime = currentTime;
      const timeElapsed = currentTime - startTime;
      const progress = Math.min(timeElapsed / duration, 1);
      window.scrollTo(0, start + distance * progress);

      if (timeElapsed < duration) {
        requestAnimationFrame(animation);
      }
    }

    requestAnimationFrame(animation);
  }

  onMount(() => {
    const navbar = document.querySelector('.navbar');

    function handleScroll() {
      if (window.scrollY > 80) {
        navbar.classList.add('shrink');
      } else {
        navbar.classList.remove('shrink');
      }
    }

    window.addEventListener('scroll', handleScroll);

    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  });
</script>

<!-- Anchor para que funcione el scroll lento -->
<div id="inicio" style="position: relative; top: -100px;"></div>

{#if futbolistas.length > 0}
<header class="navbar">
  <div class="navbar-content">
    <div class="logo-y-links">
      <!-- El botín ahora actúa como botón de 'Inicio' -->
      <div class="mini-botin-logo" on:click={() => scrollToElementoLento('inicio')} style="cursor: pointer;">
        <img class="capa" src={getBotin(futbolistas[index].continente)} alt="botin" />
        <img class="capa" src={getSuela(futbolistas[index].tiempo_en_primera)} alt="suela" />
        <img class="capa" src={getCordones(futbolistas[index].estado, futbolistas[index].gano_mundial)} alt="cordones" />
      </div>

      <nav class="nav-main-links">
        <a href="#botines" on:click|preventDefault={() => scrollToElementoLento('botines')}>Botines</a>
        <a href="#" on:click|preventDefault={() => {
          mostrarLeyenda = !mostrarLeyenda;
          if (mostrarLeyenda) {
            setTimeout(() => scrollToElementoLento('codificacion'), 100);
          }
        }}>Codificación</a>
      </nav>
    </div>

    <div class="nav-separador"></div>
    <a class="nav-destacado" href="#footer" on:click|preventDefault={() => scrollToElementoLento('footer')}>
      Visualización de Datos
    </a>
  </div>
</header>
{/if}
{#if futbolistas.length > 0}
<!-- INICIO CON FONDO -->
<section class="hero-con-fondo" id="inicio">
  <h1 class="titulo">FUTBOL CODIFICADO EN BOTINES</h1>

  <div class="carrusel" style="display: flex; justify-content: center; align-items: center; gap: 40px; margin-top: 30px;">
    <button class="carrusel-boton" on:click={() => index = (index - 1 + futbolistas.length) % futbolistas.length}>‹</button>
    
    <div class="carrusel-contenido" style="text-align: center;">
      <div class="botin-contenedor-grande">
        <img class="capa" src={getBotin(futbolistas[index].continente)} alt="botin" />
        <img class="capa" src={getSuela(futbolistas[index].tiempo_en_primera)} alt="suela" />
        <img class="capa" src={getCordones(futbolistas[index].estado, futbolistas[index].gano_mundial)} alt="cordones" />
      </div>
      <p class="nombre-grande">{futbolistas[index].nombre}</p>
    </div>

    <button class="carrusel-boton" on:click={() => index = (index + 1) % futbolistas.length}>›</button>
  </div>
</section>
{/if}

{#if mostrarLeyenda}
  <div class="leyenda" id="codificacion">
    <div class="leyenda-color">
      <h2 class="leyenda-titulo">COLOR</h2>
      <div class="fila-items">
        <div class="item-leyenda"><div class="color-circle" style="background-color: #1A9C4E;"></div><span>América</span></div>
        <div class="item-leyenda"><div class="color-circle" style="background-color: #F7A723;"></div><span>Asia</span></div>
        <div class="item-leyenda"><div class="color-circle" style="background-color: #D64123;"></div><span>África</span></div>
        <div class="item-leyenda"><div class="color-circle" style="background-color: #124E9C;"></div><span>Europa</span></div>
      </div>

      <h2 class="leyenda-titulo">SUELA</h2>
      <div class="fila-items">
        <div class="item-leyenda suela"><img src="/images/tapon_nuevo.png" /><span>Menos de 10 años</span></div>
        <div class="item-leyenda suela"><img src="/images/tapon_usado.png" /><span>10 a 17 años</span></div>
        <div class="item-leyenda suela"><img src="/images/tapon_gastado.png" /><span>Más de 17 años</span></div>
      </div>

      <h2 class="leyenda-titulo">ATADO</h2>
      <div class="fila-items atado">
        <div class="item"><img src="/images/atado_normal.png" /><span>Activo (cordones atados)</span></div>
        <div class="item"><img src="/images/desatado_normal.png" /><span>Retirado (cordones desatados)</span></div>
      </div>

      <h2 class="leyenda-titulo">PUNTA DEL CORDÓN</h2>
      <div class="fila-items">
        <div class="item"><img src="/images/atado_dorado.png" /><span>Ganó Mundial (punta dorada)</span></div>
        <div class="item"><img src="/images/atado_normal.png" /><span>No ganó Mundial (punta común)</span></div>
      </div>
    </div>
  </div>
{/if}

<!-- FRASE -->
<section id="botines">
<div class="frase-cita">
  <div class="separador-frase"></div>
  <p>
    <span class="comilla">“</span>
    Un buen par de botines y una pelota…<br>
    eso es todo lo que necesito para ser feliz.
    <span class="comilla">”</span>
  </p>
  <span class="autor">Ronaldinho</span>
</div>

<!-- GRILLA DE BOTINES -->
<div class="container">
  {#each futbolistas as f, i}
    <div class="tooltip-wrapper {i % 3 === 0 ? 'tooltip-right' : ''}">
      <div class="jugador">
        <div class="botin-contenedor">
          <img class="capa" src={getBotin(f.continente)} />
          <img class="capa" src={getSuela(f.tiempo_en_primera)} />
          <img class="capa" src={getCordones(f.estado, f.gano_mundial)} />
        </div>
        <p class="nombre">{f.nombre}</p>
      </div>

      <div class="tooltip tooltip-grid">
        <div>🌎</div><div><strong>{f.pais}</strong></div>
        <div>🎂</div><div>{f.edad} años</div>
        <div>⏱️</div><div>{f.tiempo_en_primera} años en primera</div>
        <div>📍</div><div>{f.continente}</div>
        <div>🏆</div>
        <div><span class="{f.gano_mundial === 'T' ? 'si' : 'no'}">{f.gano_mundial === 'T' ? 'Sí' : 'No'}</span> Mundial</div>
        <div>⚽</div>
        <div><span class="{f.estado === 'A' ? 'activo' : 'retirado'}">{f.estado === 'A' ? 'Activo' : 'Retirado'}</span></div>
      </div>
    </div>
  {/each}
</div>

<div class="tooltip" id="tooltip"></div>
</section>

<!-- FOOTER -->
<div class="footer-separador"></div>

<footer class="footer-limpio" id="footer">
  <div class="footer-limpio-nombres">
    <span>
      Agustín Rodríguez
      <a href="https://www.linkedin.com/in/agust%C3%ADn-rodriguez-7340b1270" target="_blank">
        <i class="fa-brands fa-linkedin footer-icon"></i>
      </a>
      <a href="https://github.com/agusrodriguezz" target="_blank">
        <i class="fa-brands fa-square-github footer-icon"></i>
      </a>
    </span>

    <span>
      Alexander Melnikov
      <a href="https://www.linkedin.com/in/alexander-melnikov" target="_blank">
        <i class="fa-brands fa-linkedin footer-icon"></i>
      </a>
      <a href="https://github.com/alexmelnikov" target="_blank">
        <i class="fa-brands fa-square-github footer-icon"></i>
      </a>
    </span>

    <span>
      Rafael Santos
      <a href="https://www.linkedin.com/in/rafael-santos-4b4463363/" target="_blank">
        <i class="fa-brands fa-linkedin footer-icon"></i>
      </a>
      <a href="https://github.com/xrafasantos" target="_blank">
        <i class="fa-brands fa-square-github footer-icon"></i>
      </a>
    </span>
  </div>

  <p class="footer-limpio-info">
    Visualización de Datos · Universidad Torcuato Di Tella · 2025
  </p>
</footer>