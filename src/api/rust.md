<style>
  .content main, #mdbook-content { padding: 0 !important; margin: 0 !important; max-width: none !important; }
  .nav-chapters { display: none; }
  #api-frame { border: none; display: block; width: 100%; }
</style>

<iframe id="api-frame" src="rust/borger/index.html"></iframe>

<script>
  const BASE = location.pathname.replace(/\/[^/]+$/, '') + '/rust/borger';
  const f = document.getElementById('api-frame');
  const resize = () => f.style.height = (window.innerHeight - f.getBoundingClientRect().top) + 'px';
  resize();
  window.addEventListener('resize', resize);

  f.addEventListener('load', () => {
    const iframePath = f.contentWindow.location.pathname + f.contentWindow.location.hash;
    history.replaceState(null, '', '#' + iframePath.replace(BASE, ''));
    resize();
  });

  if (location.hash) {
    f.src = BASE + location.hash.slice(1);
  }
</script>
