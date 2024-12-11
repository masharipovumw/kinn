<script>
  import Router from 'svelte-spa-router';
    import Page_2 from './Pages/Page_2.svelte'
    import Page_1 from './Pages/Page_1.svelte'
    import Page_3 from './Pages/Page_3.svelte'


  const routes = {
    '/': Page_1,
    '/Posivech': Page_2,
    '/Maksimov':Page_3,
  };
</script>

<style>
  header {
    background-color: #333;
    color: white;
  }

  nav a {
    margin-right: 1rem;
    color: white;
    text-decoration: none;
  }

  nav a:hover {
    text-decoration: underline;
  }
</style>

<header>
  <nav>
    <a href="/">Nazarov-Sipachev</a>
    <a href="#/Posivech">Sipachev-Posivech</a>
    <a href="#/Maksimov">Maksimov</a>
  </nav>
</header>

<main>
  <Router {routes} />
</main>
