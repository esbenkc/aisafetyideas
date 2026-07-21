<script lang="ts">
  import '../app.css';
  import { invalidate } from '$app/navigation';
  import { page } from '$app/stores';
  import { onMount } from 'svelte';
  import Logo from '$lib/components/Logo.svelte';
  let { data, children } = $props();
  let supabase = $derived(data.supabase);

  onMount(() => {
    const { data: { subscription } } = supabase.auth.onAuthStateChange((_, newSession) => {
      if (newSession?.expires_at !== data.session?.expires_at) invalidate('supabase:auth');
    });
    return () => subscription.unsubscribe();
  });
</script>

<!-- IN-DEVELOPMENT: front page stays indexable (banner shows dev status); demo-data pages are noindex. Remove this + the .dev-banner below when the site goes live -->
<svelte:head>
  {#if $page.url.pathname !== '/'}
    <meta name="robots" content="noindex" />
  {/if}
</svelte:head>

<div class="site">
  <div class="dev-banner" role="alert">
    <span class="dev-banner__tag">In development</span>
    <span class="dev-banner__msg">Demo data, not up to date. Please don't link to this as a live resource yet.</span>
  </div>

  <header class="site-header">
    <div class="site-header__inner">
      <a href="/" class="site-brand" aria-label="AI Safety Ideas - home">
        <Logo size={26} />
        <span class="site-brand__name">AI&nbsp;Safety&nbsp;Ideas</span>
      </a>
      <nav class="site-nav">
        <a href="/ideas" class="site-nav__link">Ideas</a>
        <a href="/experts" class="site-nav__link">Experts</a>
        {#if data.user}
          <a href="/dashboard" class="site-nav__link">Dashboard</a>
          {#if data.isAdmin}
            <a href="/admin" class="site-nav__link">Admin</a>
          {/if}
          <form method="POST" action="/logout" class="contents">
            <button type="submit" class="btn btn-secondary btn-sm">Sign out</button>
          </form>
        {:else}
          <a href="/login" class="btn btn-primary btn-sm">Sign in</a>
        {/if}
      </nav>
    </div>
  </header>

  <main class="site-main">{@render children()}</main>

  <footer class="site-footer">
    <div class="site-footer__inner">
      <div class="site-footer__brand">
        <a href="/" class="site-brand"><Logo size={22} /><span class="site-brand__name">AI&nbsp;Safety&nbsp;Ideas</span></a>
        <p class="site-footer__mission">
          A charitable research-bounty platform - experts post the open questions in AI safety,
          funders back them, researchers answer.
        </p>
      </div>
      <nav class="site-footer__links" aria-label="Footer">
        <a href="/ideas">Browse ideas</a>
        <a href="/experts">Experts</a>
        <a href="/login">Sign in</a>
      </nav>
      <p class="site-footer__legal">
        Donations support a 501(c)(3) charitable mission. © {new Date().getFullYear()} AI Safety Ideas.
      </p>
    </div>
  </footer>
</div>

<style>
  .dev-banner {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
    gap: 0.5rem 0.75rem;
    padding: 0.5rem 1rem;
    text-align: center;
    background: #ff9307;
    color: #1a1d1b;
    font-size: 0.82rem;
    line-height: 1.35;
    font-weight: 500;
    border-bottom: 1px solid rgba(20, 24, 22, 0.2);
  }
  .dev-banner__tag {
    flex: none;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    font-weight: 700;
    font-size: 0.68rem;
    background: rgba(20, 24, 22, 0.16);
    padding: 0.14rem 0.55rem;
    border-radius: 999px;
    white-space: nowrap;
  }
  @media (prefers-color-scheme: dark) {
    .dev-banner {
      background: #d97800;
      color: #fff;
      border-bottom-color: rgba(0, 0, 0, 0.35);
    }
    .dev-banner__tag {
      background: rgba(0, 0, 0, 0.28);
    }
  }
</style>
