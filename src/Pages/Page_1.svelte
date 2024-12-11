<script>
  import * as  XLSX from "xlsx";
  import * as d3 from "d3";

  let inputData = [];
  let activePointsCount = 11; // Default value
  let calculatedData = [];
  let showCalculatedTable = false;
  let showGraph = false;

  let sumX = 0,
    sumY = 0,
    sumXY = 0,
    sumX2 = 0,
    sumX2Squared = 0,
    A = 0,
    B = 0;

  // Handles file upload
  const handleFileUpload = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = () => {
      const data = new Uint8Array(reader.result);
      const workbook = XLSX.read(data, { type: "array" });
      const sheetName = workbook.SheetNames[0];
      const sheet = workbook.Sheets[sheetName];
      inputData = XLSX.utils.sheet_to_json(sheet, {
        header: ["Year", "Oil", "Liquid", "Waters", "WaterCut"],
      });
      inputData.forEach((row) => (row.ActivePoint = 0));
      console.log("Uploaded Data:", inputData);
    };
    reader.readAsArrayBuffer(file);
  };

  // Updates active points
  const updateActivePoints = () => {
    const startIndex = Math.max(0, inputData.length - activePointsCount);
    inputData.forEach((row, index) => {
      row.ActivePoint = index >= startIndex ? 1 : 0;
    });
    console.log("Updated Data with Active Points:", inputData);
  };

  // Calculates values for the second table
  const calculateTable = () => {
    const activeData = inputData.filter((row) => row.ActivePoint === 1);

    sumX = activeData.reduce((acc, row) => acc + (row.Waters || 0), 0);
    sumY = activeData.reduce(
      (acc, row) => acc + (row.Liquid && row.Oil ? row.Liquid / row.Oil : 0),
      0
    );
    sumXY = activeData.reduce(
      (acc, row) =>
        acc +
        ((row.Waters || 0) *
          (row.Liquid && row.Oil ? row.Liquid / row.Oil : 0)),
      0
    );
    sumX2 = activeData.reduce(
      (acc, row) => acc + Math.pow(row.Waters || 0, 2),
      0
    );

    const n = activeData.length;
    sumX2Squared = Math.pow(sumX, 2);

    // Calculate A and B
    A =
      (n * sumXY - sumX * sumY) /
      (n * sumX2 - sumX2Squared || 1); // Prevent divide by zero
    B = (sumY - A * sumX) / n;

    // Prepare calculated data for table
    calculatedData = activeData.map((row) => {
      const X = row.Waters || 0;
      const Y = row.Liquid && row.Oil ? row.Liquid / row.Oil : 0;
      return {
        Year: row.Year,
        X: X,
        Y: Y,
        XY: X * Y,
        X2: Math.pow(X, 2),
      };
    });

    showCalculatedTable = true;
    showGraph = false;
    console.log("Calculated Data:", calculatedData, { A, B });
  };

  // Render the graph
  const renderGraph = () => {
    const svgElement = document.querySelector("#graph");
    d3.select(svgElement).selectAll("*").remove();

    const width = 800,
      height = 400,
      margin = { top: 40, right: 30, bottom: 50, left: 60 };

    const xScale = d3.scaleLinear()
      .domain([0, d3.max(calculatedData, (d) => d.X)])
      .range([margin.left, width - margin.right]);

    const yScale = d3.scaleLinear()
      .domain([0, d3.max(calculatedData, (d) => d.Y)])
      .range([height - margin.bottom, margin.top]);

    const svg = d3.select(svgElement)
      .attr("width", width)
      .attr("height", height);

    svg.append("g")
      .attr("transform", `translate(0,${height - margin.bottom})`)
      .call(d3.axisBottom(xScale));

    svg.append("g")
      .attr("transform", `translate(${margin.left},0)`)
      .call(d3.axisLeft(yScale));

    svg.selectAll("circle")
      .data(calculatedData)
      .enter()
      .append("circle")
      .attr("cx", (d) => xScale(d.X))
      .attr("cy", (d) => yScale(d.Y))
      .attr("r", 5)
      .style("fill", "blue");

    svg.append("line")
      .attr("x1", xScale(0))
      .attr("y1", yScale(B))
      .attr("x2", xScale(d3.max(calculatedData, (d) => d.X)))
      .attr(
        "y2",
        yScale(A * d3.max(calculatedData, (d) => d.X) + B)
      )
      .style("stroke", "red")
      .style("stroke-width", 2);

    showGraph = true;
  };
</script>


<main>

  <section>
    <input type="file" accept=".xlsx" on:change={handleFileUpload} />
  </section>

  <section>
    <input type="number" bind:value={activePointsCount} min="1" max={inputData.length} placeholder="Enter Active Points Count" />
    <button on:click={updateActivePoints}>Update Active Points</button>
  </section>

  <section>
    <table border="1">
      <thead>
        <tr>
          <th>Year</th>
          <th>Oil</th>
          <th>Liquid</th>
          <th>Waters</th>
          <th>Water Cut</th>
          <th>Active Point</th>
        </tr>
      </thead>
      <tbody>
        {#each inputData as row}
          <tr>
            <td>{row.Year}</td>
            <td>{row.Oil}</td>
            <td>{row.Liquid}</td>
            <td>{row.Waters}</td>
            <td>{row.WaterCut}</td>
            <td>{row.ActivePoint}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </section>

  <section>
    <button on:click={calculateTable}>Calculate</button>
  </section>

  {#if showCalculatedTable}
    <section>
      <table border="1">
        <thead>
          <tr>
            <th>Year</th>
            <th>X (Water)</th>
            <th>Y (Liquid / Oil)</th>
            <th>XY</th>
            <th>X²</th>
          </tr>
        </thead>
        <tbody>
          {#each calculatedData as row}
            <tr>
              <td>{row.Year}</td>
              <td>{row.X.toFixed(2)}</td>
              <td>{row.Y.toFixed(2)}</td>
              <td>{row.XY.toFixed(2)}</td>
              <td>{row.X2.toFixed(2)}</td>
            </tr>
          {/each}
        </tbody>
      </table>
    </section>
  {/if}

  {#if showCalculatedTable}
    <section>
      <h2>Step 4: Show Graph</h2>
      <button on:click={renderGraph}>Show Graph</button>
    </section>
  {/if}

  {#if showGraph}
    <section>
      <h2>Graph</h2>
      <svg id="graph"></svg>
    </section>
  {/if}
</main>

<style>
  main {
    font-family: Arial, sans-serif;
    padding: 20px;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 20px;
  }
  table th, table td {
    padding: 8px;
    text-align: center;
  }
  button {
    padding: 10px;
    margin: 10px;
    background-color: #4CAF50;
    color: white;
    border: none;
    cursor: pointer;
  }
  button:hover {
    background-color: #45a049;
  }
</style>
