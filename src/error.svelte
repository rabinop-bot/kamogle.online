<script>
  export let error;
  export let status;
</script>

<svelte:head>
  <title>{status}: {error?.message || 'An error occurred'}</title>
</svelte:head>

<main class="error-page">
  <h1>{status}</h1>
  <p>{error?.message || 'Sorry, something went wrong.'}</p>
  <a href="/">Go back home</a>
</main>

<style>
  .error-page {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 60vh;
    color: #c00;
  }
  h1 {
    font-size: 3rem;
    margin-bottom: 1rem;
  }
  a {
    margin-top: 2rem;
    color: #0070f3;
    text-decoration: underline;
  }
</style>
