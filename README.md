
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Enhanced SIP Calculator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.7.0/chart.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
        }

        body {
            padding: 20px;
            background-color: #f5f5f5;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }

        h1 {
            color: #333;
            margin-bottom: 20px;
            text-align: center;
        }

        .input-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 20px;
        }

        .input-group {
            display: flex;
            flex-direction: column;
        }

        label {
            margin-bottom: 5px;
            color: #666;
        }

        input {
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 14px;
        }

        button {
            background-color: #2196F3;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            margin: 20px 0;
        }

        button:hover {
            background-color: #1976D2;
        }

        .results {
            margin-top: 20px;
        }

        .tabs {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }

        .tab {
            background-color: #2196F3;
             padding: 10px 20px;
            background-color: #blue;
            border: none;
            cursor: pointer;
            border-radius: 4px;
        }

        .tab.active {
            background-color: #2196F3;
            color: white;
        }

        .breakdown-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }

        .breakdown-table th,
        .breakdown-table td {
            padding: 10px;
            border: 1px solid #ddd;
            text-align: right;
        }

        .breakdown-table th {
            background-color: #f5f5f5;
        }

        .chart-container {
            margin-top: 20px;
            height: 400px;
        }

        .summary-cards {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }

        .summary-card {
            background-color: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
            text-align: center;
        }

        .summary-card h3 {
            color: #666;
            font-size: 14px;
            margin-bottom: 10px;
        }

        .summary-card p {
            color: #333;
            font-size: 18px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Enhanced SIP Calculator</h1>
        
        <div class="input-grid">
            <div class="input-group">
                <label>Monthly SIP Amount (₹):</label>
                <input type="number" id="monthlyAmount" value="10000">
            </div>
            <div class="input-group">
                <label>Expected Annual Return (%):</label>
                <input type="number" id="expectedReturn" value="10">
            </div>
            <div class="input-group">
                <label>Investment Period (Years):</label>
                <input type="number" id="investmentPeriod" value="15">
            </div>
            <div class="input-group">
                <label>Annual Top-up Rate (%):</label>
                <input type="number" id="topupRate" value="10">
            </div>
        </div>

        <button onclick="calculateSIP()">Calculate Investment</button>

        <div class="summary-cards">
            <div class="summary-card">
                <h3>Total Regular SIP Amount</h3>
                <p id="regularSIPInvestment">₹0</p>
            </div>
            <div class="summary-card">
                <h3>Regular SIP Value</h3>
                <p id="regularSIPTotal">₹0</p>
            </div>
            <div class="summary-card">
                <h3>Regular SIP Wealth Generated</h3>
                <p id="regularSIPWealth">₹0</p>
            </div>
            <div class="summary-card">
                <h3>Top-up SIP Value</h3>
                <p id="topupSIPTotal">₹0</p>
            </div>
            <div class="summary-card">
                <h3>Additional Wealth vs Regular SIP</h3>
                <p id="additionalWealth">₹0</p>
            </div>
        </div>

        <div class="results">
            <div class="tabs">
                <button class="tab active" onclick="showBreakdown('yearly')">Yearly</button>
                <button class="tab" onclick="showBreakdown('monthly')">Monthly</button>
                <!-- <button class="tab" onclick="showBreakdown('daily')">Daily</button>*/ -->
            </div>

            <div class="chart-container">
                <canvas id="sipChart"></canvas>
            </div>

            <div id="breakdownTable"></div>
        </div>
    </div>

    <script>
        let sipChart = null;
        let currentBreakdown = 'yearly';
        let calculationResults = null;

        function calculateSIP() {
            const monthlyAmount = parseFloat(document.getElementById('monthlyAmount').value);
            const expectedReturn = parseFloat(document.getElementById('expectedReturn').value);
            const investmentPeriod = parseInt(document.getElementById('investmentPeriod').value);
            const topupRate = parseFloat(document.getElementById('topupRate').value);

            const monthlyRate = expectedReturn / 12 / 100;
            let regularSIPTotal = 0;
            let topupSIPTotal = 0;
            let totalRegularInvestment = 0;
            const yearlyData = [];
            const monthlyData = [];
            const dailyData = [];

            let currentMonthlyAmount = monthlyAmount;
            let regularMonthlyAmount = monthlyAmount;

            // Calculate data for all periods
            for (let year = 1; year <= investmentPeriod; year++) {
                let yearlyRegularInvestment = regularMonthlyAmount * 12;
                totalRegularInvestment += yearlyRegularInvestment;
                let yearlyTopupInvestment = 0;
                let regularSIPValue = regularSIPTotal;
                let topupSIPValue = topupSIPTotal;

                // Calculate monthly values
                for (let month = 1; month <= 12; month++) {
                    yearlyTopupInvestment += currentMonthlyAmount;
                    
                    // Regular SIP calculations
                    regularSIPTotal = (regularSIPTotal + regularMonthlyAmount) * (1 + monthlyRate);
                    
                    // Top-up SIP calculations
                    topupSIPTotal = (topupSIPTotal + currentMonthlyAmount) * (1 + monthlyRate);

                    monthlyData.push({
                        period: `Year ${year} Month ${month}`,
                        regularSIP: {
                            investment: regularMonthlyAmount,
                            value: regularSIPTotal
                        },
                        topupSIP: {
                            investment: currentMonthlyAmount,
                            value: topupSIPTotal
                        }
                    });

                    // Daily calculations
                    const daysInMonth = 30;
                    const dailyRate = monthlyRate / daysInMonth;
                    let dailyRegularValue = regularSIPTotal;
                    let dailyTopupValue = topupSIPTotal;

                    for (let day = 1; day <= daysInMonth; day++) {
                        dailyData.push({
                            period: `Year ${year} Month ${month} Day ${day}`,
                            regularSIP: {
                                investment: regularMonthlyAmount / daysInMonth,
                                value: dailyRegularValue * (1 + dailyRate)
                            },
                            topupSIP: {
                                investment: currentMonthlyAmount / daysInMonth,
                                value: dailyTopupValue * (1 + dailyRate)
                            }
                        });
                        
                        dailyRegularValue = dailyData[dailyData.length - 1].regularSIP.value;
                        dailyTopupValue = dailyData[dailyData.length - 1].topupSIP.value;
                    }
                }

                yearlyData.push({
                    period: `Year ${year}`,
                    regularSIP: {
                        investment: yearlyRegularInvestment,
                        value: regularSIPTotal
                    },
                    topupSIP: {
                        investment: yearlyTopupInvestment,
                        value: topupSIPTotal
                    }
                });

                // Apply annual top-up
                currentMonthlyAmount *= (1 + topupRate / 100);
            }

            calculationResults = {
                yearly: yearlyData,
                monthly: monthlyData,
                daily: dailyData,
                summary: {
                    regularSIPTotal: regularSIPTotal,
                    topupSIPTotal: topupSIPTotal,
                    totalRegularInvestment: totalRegularInvestment
                }
            };

            updateUI();
        }

        function updateUI() {
            // Update summary cards
            document.getElementById('regularSIPInvestment').textContent = 
                `₹${calculationResults.summary.totalRegularInvestment.toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
            document.getElementById('regularSIPTotal').textContent = 
                `₹${calculationResults.summary.regularSIPTotal.toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
            document.getElementById('regularSIPWealth').textContent = 
                `₹${(calculationResults.summary.regularSIPTotal - calculationResults.summary.totalRegularInvestment).toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
            document.getElementById('topupSIPTotal').textContent = 
                `₹${calculationResults.summary.topupSIPTotal.toLocaleString('en-IN', {maximumFractionDigits: 0})}`;
            document.getElementById('additionalWealth').textContent = 
                `₹${(calculationResults.summary.topupSIPTotal - calculationResults.summary.regularSIPTotal).toLocaleString('en-IN', {maximumFractionDigits: 0})}`;

            showBreakdown(currentBreakdown);
        }

        function showBreakdown(period) {
            currentBreakdown = period;
            
            // Update tab states
            document.querySelectorAll('.tab').forEach(tab => {
                tab.classList.remove('active');
                if (tab.textContent.toLowerCase() === period) {
                    tab.classList.add('active');
                }
            });

            // Update chart
            updateChart(period);

            // Update table
            const data = calculationResults[period];
            let tableHTML = `
                <table class="breakdown-table">
                    <thead>
                        <tr>
                            <th>Period</th>
                            <th>Regular SIP Investment</th>
                            <th>Regular SIP Value</th>
                            <th>Regular SIP Wealth Generated</th>
                            <th>Top-up SIP Investment</th>
                            <th>Top-up SIP Value</th>
                            <th>Additional Wealth vs Regular SIP</th>
                        </tr>
                    </thead>
                    <tbody>
            `;

            let regularInvestmentTotal = 0;
            data.forEach(row => {
                regularInvestmentTotal += row.regularSIP.investment;
                const regularWealthGenerated = row.regularSIP.value - regularInvestmentTotal;
                const additionalWealth = row.topupSIP.value - row.regularSIP.value;
                tableHTML += `
                    <tr>
                        <td>${row.period}</td>
                        <td>₹${row.regularSIP.investment.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        <td>₹${row.regularSIP.value.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        <td>₹${regularWealthGenerated.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        <td>₹${row.topupSIP.investment.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        <td>₹${row.topupSIP.value.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                        <td>₹${additionalWealth.toLocaleString('en-IN', {maximumFractionDigits: 0})}</td>
                    </tr>
                `;
            });

            tableHTML += '</tbody></table>';
            document.getElementById('breakdownTable').innerHTML = tableHTML;
        }

        function updateChart(period) {
            const data = calculationResults[period];
            const ctx = document.getElementById('sipChart').getContext('2d');

            if (sipChart) {
                sipChart.destroy();
            }

            sipChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: data.map(row => row.period),
                    datasets: [
                        {
                            label: 'Regular SIP',
                            data: data.map(row => row.regularSIP.value),
                            backgroundColor: 'blue',
                            borderColor: 'blue',
                            borderWidth: 1
                        },
                        {
                            label: 'SIP with Top-up',
                            data: data.map(row => row.topupSIP.value),
                            backgroundColor: 'green',
                            borderColor: 'green',
                            borderWidth: 1
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                callback: value => '₹' + value.toLocaleString('en-IN')
                            }
                        }
                    },
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const index = context.dataIndex;
                                    const dataPoint = data[index];
                                    let regularInvestmentTotal = 0;
                                    for (let i = 0; i <= index; i++) {
                                        regularInvestmentTotal += data[i].regularSIP.investment;
                                    }
                                    
                                    if (context.datasetIndex === 0) { // Regular SIP
                                        const regularWealthGenerated = dataPoint.regularSIP.value - regularInvestmentTotal;
                                        return [
                                            `Regular SIP Value: ₹${dataPoint.regularSIP.value.toLocaleString('en-IN', {maximumFractionDigits: 0})}`,
                                            `Total Investment: ₹${regularInvestmentTotal.toLocaleString('en-IN', {maximumFractionDigits: 0})}`,
                                            `Wealth Generated: ₹${regularWealthGenerated.toLocaleString('en-IN', {maximumFractionDigits: 0})}`
                                        ];
                                    } else { // Top-up SIP
                                        const additionalWealth = dataPoint.topupSIP.value - dataPoint.regularSIP.value;
                                        return [
                                            `Top-up SIP Value: ₹${dataPoint.topupSIP.value.toLocaleString('en-IN', {maximumFractionDigits: 0})}`,
                                            `Additional Investment: ₹${(dataPoint.topupSIP.investment - dataPoint.regularSIP.investment).toLocaleString('en-IN', {maximumFractionDigits: 0})}`,
                                            `Additional Wealth vs Regular: ₹${additionalWealth.toLocaleString('en-IN', {maximumFractionDigits: 0})}`
                                        ];
                                    }
                                }
                            }
                        }
                    }
                }
            });
        }

        // Initial calculation
        calculateSIP();
    </script>
</body>
</html>
