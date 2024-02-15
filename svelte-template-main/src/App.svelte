<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    let data = [];

    onMount(async () => {

    const res = await fetch('data_full.csv'); 

    const csv = await res.text();

    data = d3.csvParse(csv, d3.autoType)

    console.log(data);
    });
    

  </script>
  
  <style>
    /* Add any styles if needed */
  </style>
  
  <div id="my_dataviz"></div>  