<svelte:head>
  <title>Webex Refresh Token Client</title>
</svelte:head>

<noscript id="javascript-warning" class="hero is-danger is-bold">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        <span class="icon">
          <i class="mdi mdi-alert" />
        </span>
        <span>Javascript is disabled.</span>
      </h1>
      <h2 class="subtitle">
        This site requires Javascript for its core functionality. Please enable Javascript in browser settings and
        reload this page.
      </h2>
    </div>
  </div>
</noscript>

<section id="cookies-warning" class="hero is-danger is-bold is-hidden">
  <div class="hero-body">
    <div class="container">
      <h1 class="title">
        <span class="icon">
          <i class="mdi mdi-alert" />
        </span>
        <span>Cookies are disabled.</span>
      </h1>
      <h2 class="subtitle">
        This site requires cookies for its core functionality. Please enable cookies in browser settings and reload this
        page.
      </h2>
    </div>
  </div>
</section>

<slot />

<style lang="scss" global>
  @use '../app.scss';
</style>
