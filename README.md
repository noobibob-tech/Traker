<!DOCTYPE html>
<html lang="en" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Finance Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        // Force dark mode class on the html element
        document.documentElement.classList.add('dark');
    </script>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; -webkit-tap-highlight-color: transparent; }
        /* Always use dark mode card styling */
        .card { @apply bg-slate-800 rounded-[20px] shadow-none border border-slate-700; }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
        .btn-sm-text { font-size: 0.875rem; padding-left: 0.75rem; padding-right: 0.75rem; }
    </style>
</head>
<body class="bg-slate-900 text-slate-100 pb-10 transition-colors duration-200">

    <!-- Header / Net Worth -->
    <div class="bg-gradient-to-br from-indigo-700 to-blue-900 text-white p-8 rounded-b-[40px] shadow-2xl mb-6 text-center">
        <p class="text-indigo-200 text-xs uppercase tracking-widest font-bold mb-1">Current Net Worth</p>
        <h1 id="netWorthDisplay" class="text-4xl font-black">₹0</h1>
        <div class="flex justify-around mt-6 pt-6 border-t border-indigo-500/30">
            <div>
                <p class="text-[10px] uppercase text-indigo-200">Cash Balance</p>
                <p id="cashInHand" class="font-bold text-lg">₹0</p>
            </div>
            <div>
                <p class="text-[10px] uppercase text-indigo-200">Remaining Debt</p>
                <p id="remainingDebt" class="font-bold text-lg">₹0</p>
            </div>
        </div>
    </div>

    <div class="px-4 space-y-6">
        
        <!-- Section 1: Income & Expenses -->
        <div class="card p-5">
            <h2 class="text-xs font-black text-slate-500 uppercase mb-4 tracking-wider">Daily Cash Flow</h2>
            <div class="space-y-3">
                <input type="text" id="entryDesc" placeholder="Description (e.g. Sales, Rent)" class="w-full bg-slate-700 p-4 rounded-2xl outline-none focus:ring-2 focus:ring-indigo-500 transition-all text-white placeholder-slate-400">
                <div class="flex gap-2">
                    <input type="number" id="entryAmt" placeholder="Amount" class="flex-1 min-w-0 bg-slate-700 p-4 rounded-2xl outline-none focus:ring-2 focus:ring-indigo-500 text-white placeholder-slate-400">
                    <button onclick="addEntry('income')" class="bg-green-600 text-white btn-sm-text rounded-2xl font-bold active:scale-90 transition-transform whitespace-nowrap">Incam</button>
                    <button onclick="addEntry('expense')" class="bg-red-600 text-white btn-sm-text rounded-2xl font-bold active:scale-90 transition-transform whitespace-nowrap">Expans</button>
                </div>
            </div>

            <div id="transactionList" class="mt-6 space-y-2 max-h-60 overflow-y-auto pr-1">
                <!-- Transactions show here -->
            </div>
        </div>

        <!-- Section 2: Loans & Repayments -->
        <div class="card p-5 border-t-4 border-amber-500">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xs font-black text-slate-500 uppercase tracking-wider">Lone & Repayments</h2>
                <button onclick="toggleLoanForm()" class="text-indigo-400 font-bold text-xs tracking-tight">+ Add New Lone</button>
            </div>
            
            <!-- Hidden form for new loans -->
            <div id="loanForm" class="hidden space-y-3 mb-6 p-4 bg-slate-700 rounded-2xl border border-slate-600">
                <input type="text" id="loanName" placeholder="Lone Name (e.g. Gold Lone)" class="w-full bg-slate-800 border border-slate-600 p-3 rounded-xl outline-none text-white placeholder-slate-400">
                <input type="number" id="loanTotal" placeholder="Total Lone Amount" class="w-full bg-slate-800 border border-slate-600 p-3 rounded-xl outline-none text-white placeholder-slate-400">
                <button onclick="addNewLoan()" class="w-full bg-indigo-600 text-white font-bold py-3 rounded-xl shadow-lg active:scale-95 transition-transform">Save Lone</button>
            </div>

            <div id="loanList" class="space-y-4">
                <!-- Loan cards show here -->
            </div>
        </div>

        <!-- Clear All Button -->
        <div class="text-center pt-4">
            <button onclick="resetData()" class="text-slate-600 text-[10px] uppercase font-bold tracking-tighter">Reset All Data</button>
        </div>
    </div>

    <!-- Modal for Repayment -->
    <div id="repayModal" class="fixed inset-0 bg-black/70 hidden flex items-center justify-center p-6 z-50 backdrop-blur-sm">
        <div class="bg-slate-800 w-full max-w-xs rounded-3xl p-6 shadow-2xl border border-slate-700">
            <h3 class="font-bold text-lg mb-2 text-white">Make Repayment</h3>
            <p id="repayTargetName" class="text-slate-400 text-sm mb-4"></p>
            <input type="number" id="repayAmtInput" placeholder="Enter Amount Paid" class="w-full bg-slate-700 p-4 rounded-xl outline-none mb-4 text-lg font-bold text-white border border-slate-600">
            <div class="flex gap-2">
                <button onclick="closeRepayModal()" class="flex-1 py-3 text-slate-400 font-bold">Cancel</button>
                <button onclick="confirmRepayment()" class="flex-1 bg-green-600 text-white py-3 rounded-xl font-bold shadow-lg active:scale-95 transition-transform">Confirm Pay</button>
            </div>
        </div>
    </div>

    <script>
        let state = {
            transactions: [],
            loans: []
        };

        let activeRepayLoan = null;

        function init() {
            const saved = localStorage.getItem('galaxy_tracker_v2');
            if (saved) state = JSON.parse(saved);
            render();
        }

        function toggleLoanForm() {
            const form = document.getElementById('loanForm');
            form.classList.toggle('hidden');
        }

        function addEntry(type) {
            const desc = document.getElementById('entryDesc').value;
            const amt = parseFloat(document.getElementById('entryAmt').value);
            if (!desc || isNaN(amt)) return;

            state.transactions.unshift({ id: Date.now(), desc, amt, type });
            saveAndRender();
            document.getElementById('entryDesc').value = '';
            document.getElementById('entryAmt').value = '';
        }

        function addNewLoan() {
            const name = document.getElementById('loanName').value;
            const total = parseFloat(document.getElementById('loanTotal').value);
            if (!name || isNaN(total)) return;

            state.loans.push({ name, total, paid: 0 });
            saveAndRender();
            document.getElementById('loanName').value = '';
            document.getElementById('loanTotal').value = '';
            toggleLoanForm();
        }

        function openRepayModal(loanName) {
            activeRepayLoan = loanName;
            document.getElementById('repayTargetName').innerText = "Paying back: " + loanName;
            document.getElementById('repayModal').classList.remove('hidden');
            document.getElementById('repayAmtInput').focus();
        }

        function closeRepayModal() {
            document.getElementById('repayModal').classList.add('hidden');
            document.getElementById('repayAmtInput').value = '';
            activeRepayLoan = null;
        }

        function confirmRepayment() {
            const amt = parseFloat(document.getElementById('repayAmtInput').value);
            if (isNaN(amt) || amt <= 0) return;

            const loan = state.loans.find(l => l.name === activeRepayLoan);
            if (loan) {
                loan.paid += amt;
                state.transactions.unshift({ 
                    id: Date.now(), 
                    desc: `Repaid: ${loan.name}`, 
                    amt: amt, 
                    type: 'expense' 
                });
            }

            saveAndRender();
            closeRepayModal();
        }

        function deleteEntry(id) {
            state.transactions = state.transactions.filter(t => t.id !== id);
            saveAndRender();
        }

        function deleteLoan(name) {
            if(confirm("Delete this lone record?")) {
                state.loans = state.loans.filter(l => l.name !== name);
                saveAndRender();
            }
        }

        function saveAndRender() {
            localStorage.setItem('galaxy_tracker_v2', JSON.stringify(state));
            render();
        }

        function render() {
            let totalInc = 0;
            let totalExp = 0;
            state.transactions.forEach(t => {
                if (t.type === 'income') totalInc += t.amt;
                else totalExp += t.amt;
            });

            const cashBal = totalInc - totalExp;

            const tList = document.getElementById('transactionList');
            tList.innerHTML = state.transactions.length ? '' : '<p class="text-center text-slate-600 text-xs py-4">No transactions yet</p>';
            state.transactions.forEach(t => {
                tList.innerHTML += `
                    <div class="flex justify-between items-center bg-slate-700/50 p-3 rounded-xl border border-slate-700">
                        <span class="text-sm font-medium text-slate-200">${t.desc}</span>
                        <div class="flex items-center gap-3">
                            <span class="font-bold text-sm ${t.type === 'income' ? 'text-green-400' : 'text-red-400'}">
                                ${t.type === 'income' ? '+' : '-'}₹${t.amt.toLocaleString()}
                            </span>
                            <button onclick="deleteEntry(${t.id})" class="text-slate-600 px-1">✕</button>
                        </div>
                    </div>
                `;
            });

            const lList = document.getElementById('loanList');
            lList.innerHTML = state.loans.length ? '' : '<p class="text-center text-slate-600 text-xs py-4">No lones added</p>';
            let totalDebt = 0;
            
            state.loans.forEach(l => {
                const balance = l.total - l.paid;
                totalDebt += balance;
                lList.innerHTML += `
                    <div class="p-4 rounded-2xl border bg-slate-700/30 border-slate-700">
                        <div class="flex justify-between items-start mb-3">
                            <h3 class="font-black text-white">${l.name}</h3>
                            <button onclick="deleteLoan('${l.name}')" class="text-slate-600 text-[10px] uppercase font-bold">Delete</button>
                        </div>
                        <div class="grid grid-cols-2 gap-4 mb-4">
                            <div><p class="text-[10px] text-slate-500 uppercase font-bold">Total Lone</p><p class="font-bold text-slate-200">₹${l.total.toLocaleString()}</p></div>
                            <div><p class="text-[10px] text-slate-500 uppercase font-bold">Repaid (Sum Payed)</p><p class="font-bold text-green-400">₹${l.paid.toLocaleString()}</p></div>
                        </div>
                        <div class="bg-slate-800 rounded-xl p-3 border border-slate-600 flex justify-between items-center">
                            <div>
                                <p class="text-[10px] text-slate-500 uppercase font-bold">True Balance</p>
                                <p class="text-lg font-black text-red-400">₹${balance.toLocaleString()}</p>
                            </div>
                            <button onclick="openRepayModal('${l.name}')" class="bg-indigo-600 text-white px-4 py-2 rounded-lg text-xs font-bold shadow-md active:scale-95 transition-transform">Pay Now</button>
                        </div>
                    </div>
                `;
            });

            document.getElementById('cashInHand').innerText = "₹" + cashBal.toLocaleString();
            document.getElementById('remainingDebt').innerText = "₹" + totalDebt.toLocaleString();
            document.getElementById('netWorthDisplay').innerText = "₹" + (cashBal - totalDebt).toLocaleString();
            
            const nwDisplay = document.getElementById('netWorthDisplay');
            if ((cashBal - totalDebt) < 0) {
                nwDisplay.classList.add('text-red-300');
            } else {
                nwDisplay.classList.remove('text-red-300');
            }
        }

        function resetData() {
            if(confirm("Are you sure? This will delete all entries!")) {
                localStorage.clear();
                state = { transactions: [], loans: [] };
                render();
            }
        }

        init();
    </script>
</body>
</html>

