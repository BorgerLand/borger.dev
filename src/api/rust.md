<!--
Access rustdocs here:
https://borger.dev/docs/api/rust/borger/index.html
-->

<style>
  .content main, #mdbook-content { padding: 0 !important; margin: 0 !important; max-width: none !important; }
  .nav-chapters { display: none; }
  #api-frame { border: none; display: block; width: 100%; }
</style>

<iframe id="api-frame" src="rust/borger/index.html"></iframe>

<script>
  const ROOT = location.pathname.replace(/\/[^/]+$/, '') + '/rust';
  const f = document.getElementById('api-frame');

  const resize = () => f.style.height = (window.innerHeight - f.getBoundingClientRect().top) + 'px';
  resize();
  window.addEventListener('resize', resize);

  const syncHash = () => {
    const iframePath = f.contentWindow.location.pathname + f.contentWindow.location.hash;
    const relative = iframePath.startsWith(ROOT) ? iframePath.slice(ROOT.length) : iframePath;
    history.replaceState(null, '', '#' + relative);
  };

  f.addEventListener('load', () => {
    syncHash();
    resize();
    f.contentWindow.addEventListener('hashchange', syncHash);
  });

  if (location.hash) {
    f.src = ROOT + location.hash.slice(1);
  }
</script>
