<script>
  import SplitPane from './components/SplitPane.svelte'
</script>

<SplitPane split='-' height={{ top: 0, btm: 200, rest: 50 }}>
  <div slot='top' id='TopLeft'>
    <div id='TopChild'></div>
  </div>
  <div slot='btm'>
    <div id='BtmChild'></div>
  </div>
</SplitPane>