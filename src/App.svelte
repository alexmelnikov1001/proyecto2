<script>
  import * as d3 from "d3";
  import futbolistas from "/src/data/futbolistas.csv";
  import { onMount, onDestroy, afterUpdate } from 'svelte';
  import html2canvas from 'html2canvas';
  import { slide } from 'svelte/transition';
  import { fly } from 'svelte/transition';

  const slides = [
  `<h3>Todo el mundo: Un juego global</h3>
   <p>Desde todos los rincones del planeta, estos botines representan la diversidad y grandeza del fútbol internacional.</p>`,

  `<h3>Sudamérica: El origen de las leyendas</h3>
   <p>Desde la pasión argentina hasta el talento brasileño, Sudamérica es cuna de cracks como Messi, Suárez y Paolo Guerrero.</p>`,

  `<h3>Medio y Norteamérica: Una nueva potencia</h3>
   <p>La región crece con figuras como Alphonso Davies y Pulisic, impulsando el futuro del fútbol en sus selecciones.</p>`,

  `<h3>Europa: Cuna de campeones</h3>
   <p>Con estrellas como Modrić, Mbappé e Iniesta, Europa domina el fútbol moderno tanto en clubes como selecciones.</p>`,

  `<h3>África: Talento imparable</h3>
   <p>África ha dado al mundo leyendas como Drogba, Salah y Mahrez, con carreras que inspiran desde Costa de Marfil hasta Egipto.</p>`,

  `<h3>Asia: Determinación y legado</h3>
   <p>Desde Japón hasta Irán, Asia aportó talento con figuras como Son, Nakata y Ali Daei —este último con ¡283 goles!</p>`,
  ];

  function loadFlourishScrolly() {
    const script = document.createElement('script');
    script.src = "https://cdn.flourish.rocks/flourish-scrolly-v3.1.0.min.js";
    script.type = "text/javascript";
    script.onload = () => initFlourishScrolly();
    document.body.appendChild(script);
  }

  onMount(() => {
    loadFlourishScrolly();
  });

  let nuevoNombre = '';
  let nuevoTiempo = '';
  let nuevoGoles = '';
  let nuevoContinente = '';
  let botinGenerado = null;

  function crearBotin() {
    if (nuevoNombre && nuevoTiempo && nuevoGoles && nuevoContinente) {
      botinGenerado = {
        nombre: nuevoNombre,
        tiempo: nuevoTiempo,
        goles: nuevoGoles,
        continente: nuevoContinente
      };
    } else {
      alert("Completá todos los campos para generar tu botín!");
    }
  }

  async function descargarBotin() {
    const node = document.getElementById("botin-card");
    const canvas = await html2canvas(node);
    const link = document.createElement("a");
    link.download = `${botinGenerado.nombre}-botin.png`;
    link.href = canvas.toDataURL();
    link.click();
  }

  let mostrarLeyenda = false;
  let index = 0;
  let direccionCarrusel = 1; // 1 = derecha, -1 = izquierda

  // Filter states
  let filters = {
    continente: '',
    estado: '',
    gano_mundial: '',
    tiempo_en_primera: '',
    goles: ''
  };

  let searchQuery = '';
  let mostrarFiltros = false;
  let ordenSeleccionada = '';

  $: visibleFutbolistas = filteredFutbolistas.filter(f =>
    f.nombre.toLowerCase().includes(searchQuery.toLowerCase())
  );

  $: if (ordenSeleccionada && visibleFutbolistas.length > 0) {
    visibleFutbolistas = [...visibleFutbolistas].sort((a, b) => {
      switch (ordenSeleccionada) {
        case 'nombre_asc':
          return a.nombre.localeCompare(b.nombre);
        case 'nombre_desc':
          return b.nombre.localeCompare(a.nombre);
        case 'goles_asc':
          return parseInt(a.goles) - parseInt(b.goles);
        case 'goles_desc':
          return parseInt(b.goles) - parseInt(a.goles);
        case 'tiempo_asc':
          return parseInt(a.tiempo_en_primera) - parseInt(b.tiempo_en_primera);
        case 'tiempo_desc':
          return parseInt(b.tiempo_en_primera) - parseInt(a.tiempo_en_primera);
        default:
          return 0;
      }
    });
  }

  $: filteredFutbolistas = futbolistas.filter(f => {
    const isInRange = (value, range) => {
      if (!range) return true;
      const num = parseInt(value);
      if (range === '1-10') return num >= 1 && num <= 10;
      if (range === '10-17') return num > 10 && num <= 17;
      if (range === '17+') return num > 17;
      if (range === '0-80') return num >= 0 && num <= 80;
      if (range === '81-200') return num > 80 && num <= 200;
      if (range === '200+') return num > 200;
      return true;
    };

    return (
      (!filters.continente || f.continente === filters.continente) &&
      (!filters.estado || f.estado === filters.estado) &&
      (!filters.gano_mundial || f.gano_mundial === filters.gano_mundial) &&
      (!filters.tiempo_en_primera || isInRange(f.tiempo_en_primera, filters.tiempo_en_primera)) &&
      (!filters.goles || isInRange(f.goles, filters.goles))
    );
  });

  function resetFilters() {
    filters = {
      continente: '',
      estado: '',
      gano_mundial: '',
      tiempo_en_primera: '',
      goles: ''
    };
  }

  let showWelcome = true;
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

    const navbar = document.querySelector('.navbar');
    const navbarHeight = navbar ? navbar.offsetHeight : 90;

    const originalStyle = target.style.cssText;
    target.style.cssText = `
      scroll-margin-top: ${navbarHeight}px;
      ${originalStyle}
    `;

    target.scrollIntoView({ behavior: 'smooth', block: 'start' });

    setTimeout(() => {
      target.style.cssText = originalStyle;
    }, 1000);
  }

  function scrollToNextSection() {
    const nextSection = document.getElementById('inicio');
    const welcomeSection = document.querySelector('.welcome-section');

    if (nextSection && welcomeSection) {
      nextSection.scrollIntoView({ behavior: 'smooth' });

      welcomeSection.classList.add('hidden');

      setTimeout(() => {
        showWelcome = false;
      }, 1600);
    }
  }

  let welcomeSectionElement;
  let leyendaModalElement;

  onMount(() => {
    const navbar = document.querySelector('.navbar');
    welcomeSectionElement = document.querySelector('.welcome-section');


    if (welcomeSectionElement) {
      const handleWheel = (event) => {
        if (showWelcome) {
          if (event.deltaY > 0) {
            event.preventDefault(); 
            scrollToNextSection();
          } else if (event.deltaY < 0) {
             event.preventDefault();
          }
        }
      };
      welcomeSectionElement.addEventListener('wheel', handleWheel);

       welcomeSectionElement._wheelListener = handleWheel;

      const scrollIndicator = welcomeSectionElement.querySelector('.scroll-indicator');
      if (scrollIndicator) {
        const handleClick = () => {
          scrollToNextSection();
        };
        scrollIndicator.addEventListener('click', handleClick);
        scrollIndicator._handleClick = handleClick;
      }
    }

    return () => {

       const scrollIndicator = welcomeSectionElement ? welcomeSectionElement.querySelector('.scroll-indicator') : null;
       if (scrollIndicator && scrollIndicator._handleClick) {
         scrollIndicator.removeEventListener('click', scrollIndicator._handleClick);
         delete scrollIndicator._handleClick;
       }

       if (welcomeSectionElement && welcomeSectionElement._wheelListener) {
          welcomeSectionElement.removeEventListener('wheel', welcomeSectionElement._wheelListener);
          delete welcomeSectionElement._wheelListener;
       }

    };
  });

  $: if (showWelcome === false) {
    window.addEventListener('scroll', handleNavbarScroll);
  } else {
    window.removeEventListener('scroll', handleNavbarScroll);
  }

  function handleNavbarScroll() {
    const navbar = document.querySelector('.navbar');

    if (navbar) {
      if (window.scrollY > 80) {
        navbar.classList.add('shrink');
      } else {
        navbar.classList.remove('shrink');
      }
    }
  }
  
  afterUpdate(() => {
    const modal = document.getElementById('codificacion');

    if (modal && mostrarLeyenda) {
      const scrollableContent = modal.querySelector('.leyenda-color');
      if (scrollableContent && !scrollableContent._wheelListener) {
        const handleModalWheel = (event) => {
        };
        scrollableContent.addEventListener('wheel', handleModalWheel, { passive: false, capture: true });
        scrollableContent._wheelListener = handleModalWheel; 
      }
    } else if (modal) { 
      const scrollableContent = modal.querySelector('.leyenda-color');
      if (scrollableContent && scrollableContent._wheelListener) {
        scrollableContent.removeEventListener('wheel', scrollableContent._wheelListener, { capture: true });
        delete scrollableContent._wheelListener;
      }
    }
  });

  $: {
    if (mostrarLeyenda) {
      document.body.classList.add('no-scroll');
    } else {
      document.body.classList.remove('no-scroll');
    }
  }

  onDestroy(() => {
     const modal = document.getElementById('codificacion');
     if (modal && modal._wheelListener) {
        modal.removeEventListener('wheel', modal._wheelListener, { capture: true });
        delete modal._wheelListener;
     }
     document.body.classList.remove('no-scroll');
  });

</script>

{#if showWelcome}
<!-- Welcome Section -->
<div class="welcome-section {mostrarLeyenda ? 'blur-content' : ''}">
  <div class="welcome-content">
    <h1 class="welcome-title">Botines que cuentan</h1>
    <p class="welcome-subtitle">Historias futboleras codificadas en cada botín.</p>
    <div class="scroll-indicator">
      <span>Click para continuar</span>
      <div class="scroll-arrow">↓</div>
    </div>
  </div>
</div>
{/if}

<div id="inicio" class="anchor-offset {mostrarLeyenda ? 'blur-content' : ''}"></div>

{#if futbolistas.length > 0}
{#if !showWelcome}
<header class="navbar {mostrarLeyenda ? 'blur-content' : ''}">
  <div class="navbar-content">
    <div class="logo-y-links">
      <div class="mini-botin-logo" on:click={() => scrollToElementoLento('inicio')} style="cursor: pointer;">
        <img class="capa" src={getBotin(futbolistas[index].continente)} alt="botin" />
        <img class="capa" src={getSuela(futbolistas[index].tiempo_en_primera)} alt="suela" />
         <img class="capa" src={getLineas(futbolistas[index].goles)} alt="lineas" />
        <img class="capa" src={getCordones(futbolistas[index].estado, futbolistas[index].gano_mundial)} alt="cordones" />
      </div>

      <nav class="nav-main-links">
        <a href="#busqueda" on:click|preventDefault={() => scrollToElementoLento('busqueda')}>Botines</a>
        <a href="#" on:click|preventDefault={() => {
          mostrarLeyenda = !mostrarLeyenda;
          if (mostrarLeyenda) {
            setTimeout(() => scrollToElementoLento('codificacion'), 100);
          }
        }}>Codificación</a>
        <a href="#crear-botin" on:click|preventDefault={() => scrollToElementoLento('crear-botin')}>Crear Botín</a>
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
        <img 
          src="/images/codificacion.jpg" 
          alt="Codificación visual" 
          style="width: 100%; max-width: 950px; height: auto; display: block; margin: 0px auto 0 auto;" 
        />
      </div>
  </div>
{/if}

<!-- INICIO CON FONDO -->

<section class="hero-con-fondo {mostrarLeyenda ? 'blur-content' : ''}" id="inicio">
  <h1 class="titulo">FUTBOL CODIFICADO EN BOTINES</h1>

  <div class="carrusel" style="display: flex; justify-content: center; align-items: center; gap: 40px; margin-top: 30px;">
    <button class="carrusel-boton" on:click={() => {
      direccionCarrusel = -1;
      index = (index - 1 + futbolistas.length) % futbolistas.length;
    }}>
      ‹
    </button>
    
    {#key index}
      <div class="carrusel-contenido" style="text-align: center;" transition:fly={{ x: 200 * direccionCarrusel, duration: 150, opacity: 0.2 }}>
        <div class="botin-contenedor-grande">
          <img class="capa" src={getBotin(futbolistas[index].continente)} alt="botin" />
          <img class="capa" src={getSuela(futbolistas[index].tiempo_en_primera)} alt="suela" />
          <img class="capa" src={getLineas(futbolistas[index].goles)} alt="goles" />
          <img class="capa" src={getCordones(futbolistas[index].estado, futbolistas[index].gano_mundial)} alt="cordones" />
        </div>
        <p class="nombre-grande">{futbolistas[index].nombre}</p>
      </div>
    {/key}

    <button class="carrusel-boton" on:click={() => {
    direccionCarrusel = 1;
    index = (index + 1) % futbolistas.length;
  }}>
    ›
  </button>
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

<div id="my-wrapper">
  <div class="flourish-embed flourish-scrolly" data-src="story/3223859"></div>

  {#each slides as slide, index}
    <div class="slide-texto">
      {@html slide}
      <a href={"#story/3223859/slide-" + (index + 1)}></a>
    </div>
  {/each}
</div>

<!-- 🔍 BUSCADOR + ORDENAR + RESETEAR -->
<div class="busqueda-wrapper" id="busqueda">
  <input
    type="text"
    placeholder="Buscar futbolista..."
    bind:value={searchQuery}
    class="input-buscador"
  />

  <select bind:value={ordenSeleccionada} class="ordenar-select">
    <option value="">Ordenar por</option>
    <option value="nombre_asc">Nombre (A → Z)</option>
    <option value="nombre_desc">Nombre (Z → A)</option>
    <option value="goles_asc">Cantidad de goles (↑)</option>
    <option value="goles_desc">Cantidad de goles (↓)</option>
    <option value="tiempo_asc">Tiempo en Primera (↑)</option>
    <option value="tiempo_desc">Tiempo en Primera (↓)</option>
  </select>

  <button class="reset-button" on:click={resetFilters}>Resetear</button>
</div>

<!-- 🧩 FILTROS -->
<div class="filtros-container">
  <div class="filtros-grid">
    <div class="filtro-grupo">
      <label>Continente</label>
      <select bind:value={filters.continente}>
        <option value="">Todos</option>
        <option value="Europa">Europa</option>
        <option value="América">América</option>
        <option value="África">África</option>
        <option value="Asia">Asia</option>
      </select>
    </div>

    <div class="filtro-grupo">
      <label>Estado</label>
      <select bind:value={filters.estado}>
        <option value="">Todos</option>
        <option value="A">Activo</option>
        <option value="R">Retirado</option>
      </select>
    </div>

    <div class="filtro-grupo">
      <label>Mundial</label>
      <select bind:value={filters.gano_mundial}>
        <option value="">Todos</option>
        <option value="T">Ganó</option>
        <option value="F">No ganó</option>
      </select>
    </div>

    <div class="filtro-grupo">
      <label>Tiempo en Primera</label>
      <select bind:value={filters.tiempo_en_primera}>
        <option value="">Todos</option>
        <option value="1-10">1-10 años</option>
        <option value="10-17">10-17 años</option>
        <option value="17+">+17 años</option>
      </select>
    </div>

    <div class="filtro-grupo">
      <label>Goles</label>
      <select bind:value={filters.goles}>
        <option value="">Todos</option>
        <option value="0-80">0-80 goles</option>
        <option value="81-200">81-200 goles</option>
        <option value="200+">+200 goles</option>
      </select>
    </div>
  </div>
</div>


<!-- GRILLA DE BOTINES -->
<div class="container">
  {#each visibleFutbolistas as f, i}
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
        <div class="tooltip-line">
          {f.pais} (<span class="continente-color {f.continente.toLowerCase()}">{f.continente}</span>)
        </div>
        <div class="tooltip-line">
          <span class="bold-number">{f.edad}</span> años
        </div>
        <div class="tooltip-line">
          <span class="bold-number">{f.tiempo_en_primera}</span> años en primera
        </div>
        <div class="tooltip-line">
          Mundial {f.gano_mundial === 'T' ? 'ganado' : 'no ganado'}
        </div>
        <div class="tooltip-line">
          <span class="bold-number">{f.goles}</span> goles
        </div>
        <div class="tooltip-line">
          {f.estado === 'A' ? 'Activo' : 'Retirado'}
        </div>
      </div>
    </div>

  {/each}
</div>

<div class="tooltip" id="tooltip"></div>
</section>

<section id="crear-botin" class="crear-botin-section">
  <h2 class="crear-titulo">Crear tu Botín</h2>
  <p class="crear-descripcion">Convertí tu historia en un botín único. Completá los campos y generalo al instante.</p>

  <div class="crear-form">
    <input class="crear-input" type="text" placeholder="Nombre" bind:value={nuevoNombre} />
    <input class="crear-input" type="number" placeholder="Tiempo en primera (años)" bind:value={nuevoTiempo} />
    <input class="crear-input" type="number" placeholder="Goles" bind:value={nuevoGoles} />
    <select class="crear-select" bind:value={nuevoContinente}>
      <option disabled selected value="">Continente</option>
      <option>Europa</option>
      <option>África</option>
      <option>Asia</option>
      <option>América</option>
    </select>
    <button class="crear-boton" on:click={crearBotin}>Generar Botín</button>
  </div>

  {#if botinGenerado}
    <div class="botin-preview-wrapper">
      <div id="botin-card" class="botin-contenedor">
        <img class="capa" src={getBotin(botinGenerado.continente)} />
        <img class="capa" src={getSuela(botinGenerado.tiempo)} />
        <img class="capa" src={getLineas(botinGenerado.goles)} />
        <img class="capa" src={getCordones("A", "F")} />
      </div>
      <p class="nombre">{botinGenerado.nombre}</p>
      <button class="descargar-btn" on:click={descargarBotin}>Descargar Botín</button>
    </div>
  {/if}
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
    <a href="https://www.linkedin.com/company/visualizaci%C3%B3n-de-datos-utdt/posts/?feedView=all" target="_blank" style="color: white; text-decoration: underline;">
      Visualización de Datos
    </a>
    ·
    <a href="https://www.utdt.edu/" target="_blank" style="color: white; text-decoration: underline;">
      Universidad Torcuato Di Tella
    </a>
    · 2025
  </p>
</footer>

{/if}