<script>
  import * as d3 from "d3";
  import futbolistas from "/src/data/futbolistas.csv";
  import { onMount, onDestroy, afterUpdate } from 'svelte';

  let mostrarLeyenda = false;
  let index = 0;

  // State variable to control visibility of welcome section
  let showWelcome = true;
  // New state variable to track welcome-to-home transition
  let isTransitioning = false;

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

  function getLineas(goles) {
    const num = parseInt(goles);
    if (num <= 50) return "/images/unalinea.png";
    if (num <= 167) return "/images/doslineas.png";
    return "/images/treslineas.png";
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

    // Get the navbar height
    const navbar = document.querySelector('.navbar');
    const navbarHeight = navbar ? navbar.offsetHeight : 90;

    // Add a temporary style to the target element to offset it
    const originalStyle = target.style.cssText;
    target.style.cssText = `
      scroll-margin-top: ${navbarHeight}px;
      ${originalStyle}
    `;

    // Scroll the element into view
    target.scrollIntoView({ behavior: 'smooth', block: 'start' });

    // Restore original style after a short delay
    setTimeout(() => {
      target.style.cssText = originalStyle;
    }, 1000);
  }

  // Function to scroll to the next section and hide welcome with a fade-out
  function scrollToNextSection() {
    const nextSection = document.getElementById('inicio');
    const welcomeSection = document.querySelector('.welcome-section');

    if (nextSection && welcomeSection) {
      // Scroll smoothly to the next section
      nextSection.scrollIntoView({ behavior: 'smooth' });

      // Add the 'hidden' class to trigger the fade-out transition
      welcomeSection.classList.add('hidden');

      // Hide the welcome section completely after the transition duration
      // The CSS transition is 1.5s, so we wait slightly longer
      setTimeout(() => {
        showWelcome = false;
      }, 1600); // Increased delay to ensure transition completes
    }
  }

  let welcomeSectionElement;
  let leyendaModalElement; // Reference to the legend modal

  onMount(() => {
    const navbar = document.querySelector('.navbar');
    welcomeSectionElement = document.querySelector('.welcome-section');

    // The navbar scroll listener will be managed reactively below

    // Handle wheel event on the welcome section when it's visible
    if (welcomeSectionElement) {
      const handleWheel = (event) => {
        // Only trigger if welcome is currently shown
        if (showWelcome) {
          if (event.deltaY > 0) { // Scrolling down
            event.preventDefault(); // Prevent default scroll
            scrollToNextSection();
          } else if (event.deltaY < 0) { // Prevent scrolling up from welcome
             event.preventDefault();
          }
        }
      };
      welcomeSectionElement.addEventListener('wheel', handleWheel);

       // Store the handler (less critical with {#if}, but safe)
       welcomeSectionElement._wheelListener = handleWheel;

      // Add click event listener to the scroll indicator
      const scrollIndicator = welcomeSectionElement.querySelector('.scroll-indicator');
      if (scrollIndicator) {
        const handleClick = () => {
          scrollToNextSection();
        };
        scrollIndicator.addEventListener('click', handleClick);
        // Store the handler
        scrollIndicator._handleClick = handleClick;
      }
    }

    // Global scroll attempt listeners are not needed if hiding the element works reliably

    // Cleanup function for onMount
    return () => {
      // No need to remove the navbar scroll listener here, as it's managed by the reactive statement.

      // The wheel listener on welcomeSectionElement is automatically removed when the element is removed from DOM.
      // The click listener needs explicit removal as its target element might not be removed if only opacity changes.
       const scrollIndicator = welcomeSectionElement ? welcomeSectionElement.querySelector('.scroll-indicator') : null;
       if (scrollIndicator && scrollIndicator._handleClick) {
         scrollIndicator.removeEventListener('click', scrollIndicator._handleClick);
         delete scrollIndicator._handleClick;
       }

      // Remove the stored handler from welcomeSectionElement if it exists
       if (welcomeSectionElement && welcomeSectionElement._wheelListener) {
          // The element might be null if showWelcome was false on mount, but we added the listener conditionally.
          // However, if it was added, it should be removed.
          // This explicit removal might not be strictly necessary due to {#if}, but doesn't hurt.
          welcomeSectionElement.removeEventListener('wheel', welcomeSectionElement._wheelListener);
          delete welcomeSectionElement._wheelListener;
       }

    };
  });

  // Reactively add or remove the scroll listener for the navbar when showWelcome changes
  $: if (showWelcome === false) {
    // Add listener when welcome is hidden (navbar appears)
    window.addEventListener('scroll', handleNavbarScroll);
  } else {
    // Remove listener when welcome is shown (navbar is hidden)
    window.removeEventListener('scroll', handleNavbarScroll);
  }

  // Function to handle navbar scroll for shrinking effect and super-shrinking past boot section
  function handleNavbarScroll() {
    const navbar = document.querySelector('.navbar');
    // Removed botinesSection as it's no longer needed for super-shrink

    if (navbar) { // Check if navbar exists before trying to modify it
      // Check for the initial shrink state
      if (window.scrollY > 80) {
        navbar.classList.add('shrink');
      } else {
        // Remove shrink when scrolled back up to the initial state
        navbar.classList.remove('shrink');
      }
    }
  }

  // Use afterUpdate to add/remove wheel listener on the modal when mostrarLeyenda changes
  // This listener prevents background scrolling when the modal is open.
  afterUpdate(() => {
    // Get the modal element using its ID for reliability
    const modal = document.getElementById('codificacion');

    if (modal && mostrarLeyenda) {
      // Modal is visible, add the wheel listener if not already added
      // Find the inner scrollable content area
      const scrollableContent = modal.querySelector('.leyenda-color');
      if (scrollableContent && !scrollableContent._wheelListener) {
        const handleModalWheel = (event) => {
          // Allow default scroll within this element (handled by overflow-y: auto)
        };
        // Use capture: true to ensure this listener runs before others
        scrollableContent.addEventListener('wheel', handleModalWheel, { passive: false, capture: true });
        scrollableContent._wheelListener = handleModalWheel; // Store reference
      }
    } else if (modal) { // Modal exists, check if it's becoming hidden
      const scrollableContent = modal.querySelector('.leyenda-color');
      if (scrollableContent && scrollableContent._wheelListener) {
        // Modal is hidden, remove the wheel listener if it exists
        scrollableContent.removeEventListener('wheel', scrollableContent._wheelListener, { capture: true });
        delete scrollableContent._wheelListener; // Clean up stored reference
      }
    }
  });

  // Reactive statement to add/remove the no-scroll class on the body
  $: {
    if (mostrarLeyenda) {
      document.body.classList.add('no-scroll');
    } else {
      document.body.classList.remove('no-scroll');
    }
  }

  // Svelte onDestroy lifecycle hook
  onDestroy(() => {
     // onMount's return function handles most cleanup.
     // Clean up the modal wheel listener if the component is destroyed while the modal is open
     const modal = document.getElementById('codificacion');
     if (modal && modal._wheelListener) {
        modal.removeEventListener('wheel', modal._wheelListener, { capture: true });
        delete modal._wheelListener;
     }
     // Also remove the no-scroll class from the body if the component is destroyed while the modal is open
     document.body.classList.remove('no-scroll');
     // Add any other listeners added outside onMount/return or afterUpdate that target elements not controlled by {#if} here if needed.
  });

</script>

{#if showWelcome}
<!-- Welcome Section -->
<div class="welcome-section {mostrarLeyenda ? 'blur-content' : ''}">
  <div class="welcome-content">
    <h1 class="welcome-title">Botines que cuentan</h1>
    <p class="welcome-subtitle">Historias futboleras codificadas en cada botín.</p>
    <div class="scroll-indicator">
      <span>Scroll para continuar</span>
      <div class="scroll-arrow">↓</div>
    </div>
  </div>
</div>
{/if}

<!-- Anchor para que funcione el scroll lento -->
<div id="inicio" style="position: relative; top: -100px;" class={mostrarLeyenda ? 'blur-content' : ''}></div>

{#if futbolistas.length > 0}
{#if !showWelcome}
<header class="navbar {mostrarLeyenda ? 'blur-content' : ''}">
  <div class="navbar-content">
    <div class="logo-y-links">
      <!-- El botín ahora actúa como botón de 'Inicio' -->
      <div class="mini-botin-logo" on:click={() => scrollToElementoLento('inicio')} style="cursor: pointer;">
        <img class="capa" src={getBotin(futbolistas[index].continente)} alt="botin" />
        <img class="capa" src={getSuela(futbolistas[index].tiempo_en_primera)} alt="suela" />
         <img class="capa" src={getLineas(futbolistas[index].goles)} alt="lineas" />
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
{#if mostrarLeyenda}
  <div class="leyenda" id="codificacion">
    <button class="close-button" on:click={() => mostrarLeyenda = false}>X</button>
    <h2 class="leyenda-main-title">CODIFICACIÓN</h2>
    <div class="leyenda-color">
      <h2 class="leyenda-titulo">Color del botín</h2>
      <div class="fila-items color-lineal">
        <div class="item-color"><div class="color-circle" style="background-color: #1A9C4E;"></div>América</div>
        <div class="item-color"><div class="color-circle" style="background-color: #F7A723;"></div>Asia</div>
        <div class="item-color"><div class="color-circle" style="background-color: #D64123;"></div>África</div>
        <div class="item-color"><div class="color-circle" style="background-color: #124E9C;"></div>Europa</div>
      </div>

      <h2 class="leyenda-titulo">Estado de la suela</h2>
      <div class="fila-items fila-suela">
        <div class="item-leyenda suela">
          <img src="/images/tapon_nuevo.png" alt="Taco nuevo" />
          <span>1-10 años</span>
        </div>
        <div class="item-leyenda suela">
          <img src="/images/tapon_usado.png" alt="Taco usado" />
          <span>10-17 años</span>
        </div>
        <div class="item-leyenda suela">
          <img src="/images/tapon_gastado.png" alt="Taco gastado" />
          <span>+17 años</span>
        </div>
      </div>

      <h2 class="leyenda-titulo">Punta del cordón</h2>
      <div class="fila-items punta">
        <div class="item"><img src="/images/punta_dorada.png" /><span>Ganó Mundial</span></div>
        <div class="item"><img src="/images/punta_gris.png" /><span>No ganó Mundial</span></div>
      </div>

      <h2 class="leyenda-titulo goles">Cantidad de goles</h2>
      <div class="fila-items fila-goles">
        <div class="item-leyenda goles">
          <img src="/images/oneline.png" alt="1 línea" />
          <span>0 – 80 goles</span>
        </div>
        <div class="item-leyenda goles">
          <img src="/images/twoline.png" alt="2 líneas" />
          <span>81 – 200 goles</span>
        </div>
        <div class="item-leyenda goles">
          <img src="/images/threeline.png" alt="3 líneas" />
          <span>+200 goles</span>
        </div>
      </div>

      <h2 class="leyenda-titulo atado">Atado del cordón</h2>
      <div class="fila-items fila-atado">
        <div class="item-leyenda atado">
          <img src="/images/nudo.png" alt="Jugador activo" />
          <span>Jugador activo</span>
        </div>
        <div class="item-leyenda atado">
          <img src="/images/no_nudo.png" alt="Jugador retirado" />
          <span>Jugador retirado</span>
        </div>
      </div>
    </div>
  </div>
{/if}

<!-- INICIO CON FONDO -->
<section class="hero-con-fondo {mostrarLeyenda ? 'blur-content' : ''}" id="inicio">
  <h1 class="titulo">FUTBOL CODIFICADO EN BOTINES</h1>

  <div class="carrusel" style="display: flex; justify-content: center; align-items: center; gap: 40px; margin-top: 30px;">
    <button class="carrusel-boton" on:click={() => index = (index - 1 + futbolistas.length) % futbolistas.length}>‹</button>
    
    <div class="carrusel-contenido" style="text-align: center;">
      <div class="botin-contenedor-grande">
        <img class="capa" src={getBotin(futbolistas[index].continente)} alt="botin" />
        <img class="capa" src={getSuela(futbolistas[index].tiempo_en_primera)} alt="suela" />
        <img class="capa" src={getLineas(futbolistas[index].goles)} alt="goles" />
        <img class="capa" src={getCordones(futbolistas[index].estado, futbolistas[index].gano_mundial)} alt="cordones" />
      </div>
      <p class="nombre-grande">{futbolistas[index].nombre}</p>
    </div>

    <button class="carrusel-boton" on:click={() => index = (index + 1) % futbolistas.length}>›</button>
  </div>
</section>

<!-- FRASE -->
<section id="botines" class={mostrarLeyenda ? 'blur-content' : ''}>
<div class="frase-cita">
  <div class="separador-frase"></div>
  <p>
    <span class="comilla">"</span>
    Un buen par de botines y una pelota…<br>
    eso es todo lo que necesito para ser feliz.
    <span class="comilla">"</span>
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
          <img class="capa" src={getSuela(f.tiempo_en_primera)} alt="suela" />
          <img class="capa" src={getLineas(f.goles)} alt="goles" />
          <img class="capa" src={getCordones(f.estado, f.gano_mundial)} alt="cordones" />
        </div>
        <p class="nombre">{f.nombre}</p>
      </div>

      <div class="tooltip tooltip-grid">
        <div class="tooltip-line">País: {f.pais}</div>
        <div class="tooltip-line">Edad: {f.edad} años</div>
        <div class="tooltip-line">Tiempo en primera: {f.tiempo_en_primera} años</div>
        <div class="tooltip-line">Continente: {f.continente}</div>
        <div class="tooltip-line">Mundial: {f.gano_mundial === 'T' ? 'Ganó' : 'No ganó'}</div>
        <div class="tooltip-line">Estado: {f.estado === 'A' ? 'Activo' : 'Retirado'}</div>
        <div class="tooltip-line">Goles: {f.goles}</div>
      </div>
    </div>

  {/each}
</div>

<div class="tooltip" id="tooltip"></div>
</section>

<!-- FOOTER -->
<div class="footer-separador {mostrarLeyenda ? 'blur-content' : ''}"></div>

<footer class="footer-limpio {mostrarLeyenda ? 'blur-content' : ''}" id="footer">
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

{/if}