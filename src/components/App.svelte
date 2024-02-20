<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";

  export let data = [];
  export let keys = [];
  export let teams = [];
  export let values = [];
  let year = 1950;
  let interval;
  let playing = false;
  let minVal = 70;
  let maxVal = 130;
  let svg;
  let x;
  let y;
  let bars;
  let height;
  let logos;
  let mouseX;
  let mouseY;

  let tooltip;

  const teamLogos = {
    "Atlanta Hawks": "modeling_plot/src/components/logos/atlanta hawks.svg",
    "Boston Celtics":
      "https://cdn.nba.com/logos/nba/1610612738/primary/L/logo.svg",
    "Brooklyn Nets":
      "https://cdn.nba.com/logos/nba/1610612751/primary/L/logo.svg",
    "New York Knicks":
      "https://cdn.nba.com/logos/nba/1610612752/primary/L/logo.svg",
    "Philadelphia 76ers":
      "https://cdn.nba.com/logos/nba/1610612755/primary/L/logo.svg",
    "Toronto Raptors":
      "https://cdn.nba.com/logos/nba/1610612761/primary/L/logo.svg",
    "Chicago Bulls":
      "https://cdn.nba.com/logos/nba/1610612741/primary/L/logo.svg",
    "Cleveland Cavaliers":
      "https://cdn.nba.com/logos/nba/1610612739/primary/L/logo.svg",
    "Detroit Pistons":
      "https://cdn.nba.com/logos/nba/1610612765/primary/L/logo.svg",
    "Indiana Pacers":
      "https://cdn.nba.com/logos/nba/1610612754/primary/L/logo.svg",
    "Milwaukee Bucks":
      "https://cdn.nba.com/logos/nba/1610612749/primary/L/logo.svg",
    "Atlanta Hawks":
      "https://cdn.nba.com/logos/nba/1610612737/primary/L/logo.svg",
    "Charlotte Hornets":
      "https://cdn.nba.com/logos/nba/1610612766/primary/L/logo.svg",
    "Miami Heat": "https://cdn.nba.com/logos/nba/1610612748/primary/L/logo.svg",
    "Orlando Magic":
      "https://cdn.nba.com/logos/nba/1610612753/primary/L/logo.svg",
    "Washington Wizards":
      "https://cdn.nba.com/logos/nba/1610612764/primary/L/logo.svg",
    "Denver Nuggets":
      "https://cdn.nba.com/logos/nba/1610612743/primary/L/logo.svg",
    "Minnesota Timberwolves":
      "https://cdn.nba.com/logos/nba/1610612750/primary/L/logo.svg",
    "Oklahoma City Thunder":
      "https://cdn.nba.com/logos/nba/1610612760/primary/L/logo.svg",
    "Portland Trail Blazers":
      "https://cdn.nba.com/logos/nba/1610612757/primary/L/logo.svg",
    "Utah Jazz": "https://cdn.nba.com/logos/nba/1610612762/primary/L/logo.svg",
    "Golden State Warriors":
      "https://cdn.nba.com/logos/nba/1610612744/primary/L/logo.svg",
    "Los Angeles Clippers":
      "https://cdn.nba.com/logos/nba/1610612746/primary/L/logo.svg",
    "Los Angeles Lakers":
      "https://cdn.nba.com/logos/nba/1610612747/primary/L/logo.svg",
    "Phoenix Suns":
      "https://cdn.nba.com/logos/nba/1610612756/primary/L/logo.svg",
    "Sacramento Kings":
      "https://cdn.nba.com/logos/nba/1610612758/primary/L/logo.svg",
    "Dallas Mavericks":
      "https://cdn.nba.com/logos/nba/1610612742/primary/L/logo.svg",
    "Houston Rockets":
      "https://cdn.nba.com/logos/nba/1610612745/primary/L/logo.svg",
    "Memphis Grizzlies":
      "https://cdn.nba.com/logos/nba/1610612763/primary/L/logo.svg",
    "New Orleans Pelicans":
      "https://cdn.nba.com/logos/nba/1610612740/primary/L/logo.svg",
    "San Antonio Spurs":
      "https://cdn.nba.com/logos/nba/1610612759/primary/L/logo.svg",
  };

  onMount(() => {
    d3.csv("src/DSC106_NBA.csv").then((csvData) => {
      data = csvData;
      renderBarChart(year);
    });
  });

  function renderBarChart(year) {
    teams = data.filter((d) => d.Year == year);
    keys = Object.keys(teams[0]).filter((key) => key !== "Year");
    values = Object.entries(teams[0])
      .filter((entry) => entry[0] !== "Year")
      .map(([key, value]) => {
        const logo = teamLogos[key];
        return {
          team: key,
          score: value,
          logo:
            logo ||
            "https://cdn.freebiesupply.com/images/large/2x/nba-logo-transparent.png",
        }; // Provide a default logo URL if not found
      });

    const margin = { top: 40, right: 120, bottom: 150, left: 60 };
    const width = 1400 - margin.left - margin.right;
    height = 600 - margin.top - margin.bottom;

    svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    x = d3.scaleBand().domain(keys).range([0, width]).padding(0.1);

    y = d3.scaleLinear().domain([minVal, maxVal]).range([height, 0]);

    svg
      .append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x))
      .selectAll("text")
      .attr("transform", "rotate(-45)")
      .style("text-anchor", "end");

    svg.append("g").call(d3.axisLeft(y));

    svg
      .selectAll(".logo")
      .data(values)
      .enter()
      .append("svg:image")
      .attr("xlink:href", (d) => d.logo)
      .attr("x", (d) => x(d.team) + x.bandwidth() / 2 - 15) // Adjust positioning as needed
      .attr("y", (d) => y(d.score) - 40) // Adjust positioning as needed
      .attr("width", 30)
      .attr("height", 30)
      .attr("class", "logo");

    svg
      .selectAll("rect")
      .data(values)
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.team))
      .attr("y", (d) => y(d.score))
      .attr("width", x.bandwidth())
      .attr("height", (d) => height - y(d.score))
      .attr("fill", "#FA8320")
      .attr("stroke", "black")
      .attr("stroke-width", 2)
      .on("mouseover", (event, d) => {
        const tooltipWidth = 120;
        const tooltipHeight = 60;

        const tooltip = svg
          .append("g")
          .attr("class", "tooltip")
          .attr(
            "transform",
            `translate(${x(d.team) + x.bandwidth()}, ${y(d.score)})`,
          );

        tooltip
          .append("rect")
          .attr("width", tooltipWidth)
          .attr("height", tooltipHeight)
          .attr("fill", "rgba(255, 255, 255, 0.8)")
          .attr("stroke", "black")
          .attr("stroke-width", 1);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 - 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`${d.team}`);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 + 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`PPG: ${d.score}`);
      })
      .on("mousemove", (event, d) => {
        updateTooltipPosition(event, d);
      })
      .on("mouseout", (event) => {
        svg.select(".tooltip").remove();
      });
    svg
      .append("text")
      .attr("transform", `translate(${width / 2}, ${height + 120})`) // Adjust the position as needed
      .style("text-anchor", "middle")
      .text("Teams")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      );

    svg
      .append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left) // Adjust the position as needed
      .attr("x", 0 - height / 2)
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Average Points Per Game")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      );
  }

  function updateTooltipPosition(event, d) {
    tooltip = svg.select(".tooltip");
    const tooltipWidth = 100;
    const tooltipHeight = 40;
    const mouseX = event.pageX;
    const mouseY = event.pageY;

    // tooltip.attr(
    //   "transform",
    //   `translate(${mouseX - 280}, ${mouseY - tooltipHeight - 170})`,
    // );
    // tooltip.attr(
    //   "transform",
    //   `translate(${0.5 * mouseX}, ${0.5 * mouseY})`,
    // );

    // tooltip.attr('transform', `translate(${mouseX-40}, ${mouseY - tooltipHeight - 10})`);
    // tooltip.select('.text').text(`${event.srcElement.__data__.team}: ${event.srcElement.__data__.score} dsfjhsdk`);
    // console.log(tooltip.select('.text'))
    // console.log(event.srcElement.__data__);
    // console.log(tooltip.selectAll('text'));
    // const first = tooltip.select('text')
    // tooltip.select('text').text(`${d.team}: ${d.score}`);
    tooltip.select("text:nth-child(3)").text(`PPG: ${d.score}`);
    // tooltip.selectAll('text').text(`${d.team}: ${d.score}`);
  }

  function updateBars(newyr) {
    teams = data.filter((d) => d.Year == newyr);
    keys = Object.keys(teams[0]).filter((key) => key !== "Year");
    values = Object.entries(teams[0])
      .filter((entry) => entry[0] !== "Year")
      .map(([key, value]) => {
        return { team: key, score: value };
      });

    bars = svg.selectAll("rect").data(values);

    bars
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.team))
      .attr("width", x.bandwidth())
      .merge(bars)
      .transition()
      .duration(800)
      .attr("y", (d) => y(d.score))
      .attr("height", (d) => height - y(d.score))
      .attr("fill", "#FA8320")
      .attr("stroke", "black")
      .attr("stroke-width", 2);

    logos = svg.selectAll(".logo").data(values);
    logos
      .enter()
      .append("svg:image")
      .attr("xlink:href", (d) => d.logo)
      .attr("x", (d) => x(d.team) + x.bandwidth() / 2 - 15) // Adjust positioning as needed
      .merge(logos)
      .transition()
      .duration(800)
      .attr("y", (d) => y(d.score) - 40)
      .attr("height", (d) => y(d.score) - 800)
      .attr("class", "logo")
      .on("mouseover", (event, d) => {
        tooltip = svg.append("g").attr("class", "tooltip");

        tooltip
          .append("rect")
          .attr("width", 130)
          .attr("height", 60)
          .attr("fill", "rgba(255, 255, 255, 0.8)")
          .attr("stroke", "black")
          .attr("stroke-width", 1);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 - 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`${d.team}`);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 + 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`PPG: ${d.score}`);

        updateTooltipPosition(event, d);
      })
      .on("mousemove", (event, d) => {
        updateTooltipPosition(event, d);
      })
      .on("mouseout", (event) => {
        svg.select(".tooltip").remove();
      });

    bars.exit().remove();
  }

  function updateYear(newYear) {
    year = newYear;
    updateBars(year);
  }

  function play() {
    playing = true;
    interval = setInterval(() => {
      if (year < 2022) {
        year++;
        updateBars(year);
      } else {
        stop();
      }
    }, 1000);
  }

  function stop() {
    clearInterval(interval);
    playing = false;
  }
</script>

<main>
  <div id="chart">
    <h1>
      Is Defense Dying in the NBA?<img
        src="https://images.ctfassets.net/h8q6lxmb5akt/5qXnOINbPrHKXWa42m6NOa/421ab176b501f5bdae71290a8002545c/nba-logo_2x.png"
        ,
        alt="NBA"
      />
    </h1>
    <h2 style="text-align: left;">NBA Teams Average Points per Game in {year}</h2>
    <div id="overlay">
      <!-- svelte-ignore a11y-label-has-associated-control -->
      <label>{year}</label>
      <input
        type="range"
        min="1950"
        max="2022"
        value={year}
        on:input={(e) => updateYear(+e.target.value)}
      />
      {#if playing}
        <button on:click={stop}>Stop</button>
      {:else}
        <button on:click={play}>Play</button>
      {/if}
    </div>
  </div>
</main>

<style>
  #overlay {
    font-size: 0.9em;
    /* background-color: rgba(157, 152, 152, 0.2); */
    position: absolute;
    min-width: 250px;
    width: 15%;
    top: 180px;
    right: 10px;
    padding: 10px;
    z-index: 3;
  }
  input {
    display: inline-block;
    width: 100%;
    position: relative;
    margin: 0;
    cursor: pointer;
  }
  label {
    font-size: 1.5em;
    font-family: sans-serif;
    font-weight: bold;
  }
  #chart {
    background-color: antiquewhite;
    margin: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
  }

  h2 {
    font-size: 24px;
    /* color: #FA8320; */
    font-family: Impact, Haettenschweiler, "Arial Narrow Bold", sans-serif;
  }

  h1 {
    font-size: 36px;
    font-family: Impact, Haettenschweiler, "Arial Narrow Bold", sans-serif;
    /* color: #FA8320; */
  }
  img {
    position: absolute;
    top: 0;
    left: 0;
    height: 20%;
  }
</style>
