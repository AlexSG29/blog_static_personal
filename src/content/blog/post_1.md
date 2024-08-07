---
title: 'Implementación manual del Modo Oscuro/Claro con Astro y TailwindCSS'
description: 'Cómo crear un botón personalizado que permita a los usuarios cambiar entre modos oscuro y claro en tu sitio web usando Astro y TailwindCSS.'
pubDate: 'Aug 03 2024'
heroImage: '/darkmode-tailwind-astro.png'
---
En este post, exploraremos cómo agregar un botón para cambiar entre modo oscuro y modo claro en un proyecto de Astro utilizando TailwindCSS. También veremos cómo manejar la persistencia del tema con `localStorage` y cómo integrarlo cuando se usa la funcionalidad de View Transitions para una navegación entre varias páginas del proyecto.

##### Paso 1: Configura el Toggle de Tema

Primero, agregaremos un botón que hara de toggle para alternar entre el modo oscuro y claro. Aquí está el código HTML para el botón en el encabezado de tu página:

```html
---
<!-- importamos los componentes que contienen los svg -->
import MoonIcon from ...
import SunIcon from ...
<!-- Es su decision elegir los iconos que 
 representaran el modo oscuro y el modo claro -->
---
<!-- simulamos una estructura basica de un header -->
<header >
  <div>
    <a>seccion 1</a>
    <a>seccion 2</a>
    <a>Seccion 3</a>
    <button id="themeToggle">
      <div id="moon"><MoonIcon /></div>
      <div id="sun" style="display: none;"><SunIcon /></div>
    </button>
  </div>
</header>
```

aqui lo importante es poder identificar los componentes que luego vamos a necesitar como seria el boton, y ambos componentes
que representan lo iconos, por ello usamos los "id" que luego llamaremos en nuestro script.

##### Paso 2: Establecer el data-theme en el html

En nuestro archivo `tailwind.config.mjs`debemos poner la siguiente configuracion: `darkMode: ['class', '[data-theme="dark"]'],`
de esta manera la propiedad data-theme sera la encargada de ser la que manera del dark mode de tailwind por defecto.
Es decir, cuando el data-theme este en "dark" se aplicara el modo oscuro, y cuando no tenga esta propiedad, no se aplicaran los estilos de dark mode de tailwind.

##### Paso 3: Implementacion del script que maneja el tema

Ahora agregaremos el script para manejar el cambio de tema. Este script se encargará de leer el tema almacenado en localStorage, aplicar el tema correspondiente, y actualizar el icono del botón. Para aplicar el tema, al estar trabajando con Tailwind CSS
lo unico que tenemo que poner es hacer que el data-theme de la etiqueta `html`cambie de "dark" a "ligth".

```html
<script>
   document.addEventListener('astro:page-load', () => {
    const theme = (() => {
      if (typeof localStorage !== 'undefined' && localStorage.getItem('theme')) {
        return localStorage.getItem('theme');
      }
      if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
        return 'dark';
      }
      return 'light';
    })();

    document.documentElement.setAttribute('data-theme', theme);
    updateIcon(theme);

    const handleToggleClick = () => {
      const element = document.documentElement;
      const currentTheme = element.getAttribute('data-theme');
      const newTheme = currentTheme === "dark" ? "light" : "dark";

      element.setAttribute('data-theme', newTheme);
      localStorage.setItem("theme", newTheme);
      updateIcon(newTheme);
    }

    document.getElementById("themeToggle").addEventListener("click", handleToggleClick);

    function updateIcon(theme) {
      const moonIcon = document.getElementById('moon');
      const sunIcon = document.getElementById('sun');

      if (theme === 'dark') {
        moonIcon.style.display = 'block';
        sunIcon.style.display = 'none';
      } else {
        moonIcon.style.display = 'none';
        sunIcon.style.display = 'block';
      }
    }
  });
</script>
```

Este script configura el tema inicial, actualiza el icono según el tema actual, y maneja el cambio de tema cuando se hace clic en el botón. Aqui es importante encapsular todo el script dentro de `document.addEventListener('astro:page-load', () => {});`
esto para que pueda funcionar cuando se añade los `<ViewTransition />` en el header de nuestro proyecto, pues
ahora los scripts se tienen que comunicar entre si en varias páginas y funcionar de igual manera.

##### Paso 4 Asegurar la Persistencia del Tema con View Transitions

Finalmente, para aplicar el tema correctamente después de una transición de página y que no se presenten los flash del color
anterior del tema, añade el siguiente script:

```html
<script>
  document.addEventListener('astro:after-swap', () => {
    localStorage.theme === 'dark'
    ? document.documentElement.classList.add("dark")
    : document.documentElement.classList.remove("dark");
  });
</script>
```

Este script asegura que el tema correcto se aplique después de la carga o intercambio de la página.

##### Nota final

En este ejemplo, no hemos utilizado is:inline en los scripts, ya que estamos aprovechando las funcionalidades de View Transitions y otros métodos de gestión del tema. Las View Transitions permiten una experiencia de usuario más fluida al realizar transiciones entre estados de la página, y la implementación del toggle de modo oscuro se maneja a través de localStorage y ajustes en el DOM. Esta aproximación asegura una transición visualmente atractiva y un manejo adecuado del estado del tema sin necesidad de is:inline
