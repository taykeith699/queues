
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bank Queue System</title>

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #f1f5f9;
  color: #1e293b;
}

header {
  background: #073b70;
  color: white;
  padding: 24px;
  text-align: center;
}

.container {
  max-width: 1000px;
  margin: 25px auto;
  padding: 0 16px;
}

.stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.card {
  background: white;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 20px;
  box-shadow: 0 3px 10px #0001;
}

.stat { text-align: center; }

.stat h2 {
  font-size: 30px;
  color: #0756a0;
}

input, select, button {
  padding: 12px;
  border-radius: 7px;
  font-size: 15px;
}

input, select {
  width: 100%;
  border: 1px solid #cbd5e1;
  margin-bottom: 10px;
}

button {
  border: none;
  cursor: pointer;
  color: white;
  background: #0756a0;
  margin: 4px 2px;
}

button:hover { opacity: .85; }
.green { background: #15803d; }
.orange { background: #d97706; }
.red { background: #dc2626; }

.ticket {
  color: #0756a0;
  font-size: 38px;
  font-weight: bold;
}

.table-wrap { overflow-x: auto; }

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 12px 8px;
  border-bottom: 1px solid #e2e8f0;
  text-align: left;
}

th { background: #f8fafc; }

.status {
  background: #fef3c7;
  padding: 5px 8px;
  border-radius: 12px;
  font-size: 12px;
}

footer {
  text-align: center;
  padding: 20px;
  color: #64748b;
}

@media(max-width:600px) {
  .stats { grid-template-columns: 1fr; }
}
</style>
</head>

<body>
<header>
  <h1>Bank Queue Management System</h1>
  <p>Customer Service Dashboard</p>
</header>

<main class="container">

  <section class="stats">
    <div class="card stat">
      <p>Now Serving</p>
      <h2 id="nowServing">---</h2>
    </div>

    <div class="card stat">
      <p>Customers Waiting</p>
      <h2 id="waitingCount">0</h2>
    </div>

    <div class="card stat">
      <p>Customers Served</p>
      <h2 id="servedCount">0</h2>
    </div>
  </section>

  <section class="card">
    <h2>Register Customer</h2>

    <form id="customerForm">
      <input id="customerName"
        placeholder="Enter customer name"
        maxlength="80" required>

      <select id="serviceType" required>
        <option value="">Select service</option>
        <option>Deposit</option>
        <option>Withdrawal</option>
        <option>Money Transfer</option>
        <option>Account Opening</option>
        <option>Customer Support</option>
      </select>

      <button type="submit">Add to Queue</button>
    </form>
  </section>

  <section class="card">
    <h2>Teller Control Panel</h2>

    <p>Current Ticket</p>
    <div class="ticket" id="currentTicket">None</div>
    <p id="currentDetails">No customer being served.</p>

    <button id="nextBtn">Call Next Customer</button>
    <button id="serveBtn" class="green">Mark as Served</button>
    <button id="skipBtn" class="orange">Skip Customer</button>
    <button id="resetBtn" class="red">Reset Demo Data</button>
  </section>

  <section class="card">
    <h2>Customers Waiting</h2>
    <div class="table-wrap">
      <table>
        <thead>
          <tr>
            <th>Ticket</th>
            <th>Customer</th>
            <th>Service</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody id="queueTable"></tbody>
      </table>
    </div>
  </section>

  <section class="card">
    <h2>Service History</h2>
    <div class="table-wrap">
      <table>
        <thead>
          <tr>
            <th>Ticket</th>
            <th>Customer</th>
            <th>Service</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody id="historyTable"></tbody>
      </table>
    </div>
  </section>

</main>

<footer>Bank Queue System | Educational Demonstration</footer>

<script>
const STORAGE_KEY = "bankQueueDemoV2";

function createDemoData() {
  return {
    nextNumber: 9,
    servedCount: 2,
    current: null,

    queue: [
      {number: 3, name: "Ama Mensah",
       service: "Deposit", status: "Waiting"},

      {number: 4, name: "Kwame Asante",
       service: "Withdrawal", status: "Waiting"},

      {number: 5, name: "Abena Owusu",
       service: "Money Transfer", status: "Waiting"},

      {number: 6, name: "Kofi Boateng",
       service: "Account Opening", status: "Waiting"},

      {number: 7, name: "Akosua Addo",
       service: "Customer Support", status: "Waiting"},

      {number: 8, name: "Yaw Osei",
       service: "Deposit", status: "Waiting"}
    ],

    history: [
      {number: 2, name: "Esi Arthur",
       service: "Withdrawal", status: "Served"},

      {number: 1, name: "Kojo Appiah",
       service: "Deposit", status: "Served"}
    ]
  };
}

function loadData() {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    return saved ? JSON.parse(saved) : createDemoData();
  } catch (error) {
    return createDemoData();
  }
}

let data = loadData();

function saveData() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
  } catch (error) {
    alert("Could not save data in this browser.");
  }
}

function ticket(number) {
  return "A" + String(number).padStart(3, "0");
}

function addCell(row, value) {
  const cell = document.createElement("td");
  cell.textContent = value;
  row.appendChild(cell);
}

function makeRow(customer) {
  const row = document.createElement("tr");

  addCell(row, ticket(customer.number));
  addCell(row, customer.name);
  addCell(row, customer.service);

  const cell = document.createElement("td");
  const badge = document.createElement("span");
  badge.className = "status";
  badge.textContent = customer.status;
  cell.appendChild(badge);
  row.appendChild(cell);

  return row;
}

function renderTable(id, customers, emptyMessage) {
  const table = document.getElementById(id);
  table.replaceChildren();

  if (customers.length === 0) {
    const row = document.createElement("tr");
    const cell = document.createElement("td");

    cell.colSpan = 4;
    cell.textContent = emptyMessage;
    row.appendChild(cell);
    table.appendChild(row);
    return;
  }

  customers.forEach(customer => {
    table.appendChild(makeRow(customer));
  });
}

function render() {
  document.getElementById("nowServing").textContent =
    data.current ? ticket(data.current.number) : "---";

  document.getElementById("waitingCount").textContent =
    data.queue.length;

  document.getElementById("servedCount").textContent =
    data.servedCount;

  document.getElementById("currentTicket").textContent =
    data.current ? ticket(data.current.number) : "None";

  document.getElementById("currentDetails").textContent =
    data.current
      ? data.current.name + " — " + data.current.service
      : "No customer being served.";

  renderTable("queueTable", data.queue, "No customers waiting.");
  renderTable("historyTable", data.history, "No service history.");

  saveData();
}

// Add customer to the rear of the queue.
document.getElementById("customerForm")
  .addEventListener("submit", function(event) {
    event.preventDefault();

    const name =
      document.getElementById("customerName").value.trim();

    const service =
      document.getElementById("serviceType").value;

    if (!name || !service) return;

    data.queue.push({
      number: data.nextNumber++,
      name: name,
      service: service,
      status: "Waiting"
    });

    this.reset();
    render();
  });

// Call the first customer in the queue.
document.getElementById("nextBtn")
  .addEventListener("click", function() {
    if (data.current) {
      alert("Finish or skip the current customer first.");
      return;
    }

    if (data.queue.length === 0) {
      alert("No customers are waiting.");
      return;
    }

    data.current = data.queue.shift();
    data.current.status = "Being Served";
    render();
  });

// Mark current customer as served.
document.getElementById("serveBtn")
  .addEventListener("click", function() {
    if (!data.current) {
      alert("Call a customer first.");
      return;
    }

    data.current.status = "Served";
    data.history.unshift(data.current);
    data.servedCount++;
    data.current = null;
    render();
  });

// Return skipped customer to the rear.
document.getElementById("skipBtn")
  .addEventListener("click", function() {
    if (!data.current) {
      alert("There is no current customer.");
      return;
    }

    data.current.status = "Waiting";
    data.queue.push(data.current);
    data.current = null;
    render();
  });

// Restore the original sample records.
document.getElementById("resetBtn")
  .addEventListener("click", function() {
    if (!confirm("Restore the original demonstration data?")) {
      return;
    }

    data = createDemoData();
    render();
  });

render();
</script>
</body>
</html>
