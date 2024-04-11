# App.py
Declares an accounting database to be used
```
db = SQL("sqlite:///accounting.db")
```

Lists all of the things that will be used in the accounting database
```
ACCOUNTS = [
    "Cash",
    "Accounts Receivable",
    "Supplies",
    "Prepaid Insurance",
    "Prepaid Rent",
    "Vehicle",
    "Land",
    "Building",
    "Acc. Depreciation",
    "Accounts Payable",
    "Unearned Revenue",
    "Wages Payable",
    "Owner Capital",
    "Owner Drawing",
    "Revenue",
    "Supplies Expense",
    "Depreciation Expense",
    "Insurance Expense",
    "Wages Expense",
    "Rent Expense",
    "Electricity Expense",
    "Water Expense",
    "Internet Expense",
]
JOURNALS = ["General Journal", "Adjustment Journal"]

Asset = [
    "Cash",
    "Accounts Receivable",
    "Supplies",
    "Prepaid Insurance",
    "Prepaid Rent",
    "Vehicle",
    "Land",
    "Building",
]

Asset_Depreciation = ["Acc. Depreciation"]

Liabilities = ["Accounts Payable", "Unearned Revenue", "Wages Payable"]

Equity = ["Owner Capital", "Owner Drawing"]

Revenue = ["Revenue"]

Expense = [
    "Supplies Expense",
    "Depreciation Expense",
    "Insurance Expense",
    "Wages Expense",
    "Rent Expense",
    "Electricity Expense",
    "Water Expense",
    "Internet Expense",
]
```

Renders an index.html template as a homepage
```
@app.route("/")
def index():
    return render_template("index.html")
```

Gets date input, debit account input, credit account input, debit value input, and credit value input
```
@app.route("/input", methods=["GET", "POST"])
def input():
    if request.method == "POST":
        date = request.form.get("transaction_date")
        debit_account = request.form.get("debit_account")
        credit_account = request.form.get("credit_account")
        debit_value = request.form.get("debit_value")
        credit_value = request.form.get("credit_value")
```

Matches the debit account input and the debit account list
```
if debit_account in Asset:
    debit_type = "Asset"
elif debit_account in Asset_Depreciation:
    debit_type = "Asset_Depreciation"
elif debit_account in Liabilities:
    debit_type = "Liabilities"
elif debit_account in Equity:
    debit_type = "Equity"
elif debit_account in Revenue:
    debit_type = "Revenue"
elif debit_account in Expense:
    debit_type = "Expense"
```

Matches the credit account input and the credit account list
```
if credit_account in Asset:
    credit_type = "Asset"
elif credit_account in Asset_Depreciation:
    credit_type = "Asset_Depreciation"
elif credit_account in Liabilities:
    credit_type = "Liabilities"
elif credit_account in Equity:
    credit_type = "Equity"
elif credit_account in Revenue:
    credit_type = "Revenue"
elif credit_account in Expense:
    credit_type = "Expense"
```

Inserts the current date when the general or adjustment journal is inputted and gets its ID
```
if request.form.get("journal") == "General Journal":
    db.execute("INSERT INTO General_Journal (Date) VALUES (?)", date)
else:
    db.execute("INSERT INTO Adjustment_Journal (Date) VALUES (?)", date)

id_row = db.execute("SELECT last_insert_rowid() AS id")
id = id_row[0]["id"]
```

Inserts all of the inputted data into the general or adjustment journal
```
if request.form.get("journal") == "General Journal":
    db.execute(
        "INSERT INTO Debit_Account (DA_Name, Type, Debit, GJ_ID) VALUES (?, ?, ?, ?)",
        debit_account,
        debit_type,
        debit_value,
        id,
    )

    db.execute(
        "INSERT INTO Credit_Account (CA_Name, Type, Credit, GJ_ID) VALUES (?, ?, ?, ?)",
        credit_account,
        credit_type,
        credit_value,
        id,
    )

else:
    db.execute(
        "INSERT INTO Debit_Account (DA_Name, Type, Debit, AJ_ID) VALUES (?, ?, ?, ?)",
        debit_account,
        debit_type,
        debit_value,
        id,
    )

    db.execute(
        "INSERT INTO Credit_Account (CA_Name, Type, Credit, AJ_ID) VALUES (?, ?, ?, ?)",
        credit_account,
        credit_type,
        credit_value,
        id,
    )
```

Renders the input.html
```
return redirect("/input")
    if request.method == "GET":
        return render_template("input.html", accounts=ACCOUNTS, journals=JOURNALS)
```

Checks if the month is between 1 (January) and 12 (December)
```
@app.route("/general", methods=["GET", "POST"])
def general_journal():
    if request.method == "POST":
        if 1 > int(request.form.get("month")) > 12:
            return render_template("general.html")
```

Displays the content of the general journal based on the month and year inputted
```
        month = request.form.get("month")
        month = "0" + month
        year = request.form.get("year")
        journal = db.execute(
            "SELECT Date, DA_NAME, CA_NAME, Debit, Credit FROM Credit_Account JOIN Debit_Account ON Credit_Account.GJ_ID=Debit_Account.GJ_ID JOIN General_Journal ON Credit_Account.GJ_ID=General_Journal.GJ_ID WHERE strftime('%m', Date) = ? AND strftime('%Y', Date) = ? ORDER BY Date",
            month,
            year,
        )

        return render_template("general_journal.html", journal=journal)
```

Renders the general.html
```
    if request.method == "GET":
        return render_template("general.html")
```

Checks if the month is between 1 (January) and 12 (December)
```
@app.route("/adjustment", methods=["GET", "POST"])
def adjustment_journal():
    if request.method == "POST":
        if 1 > int(request.form.get("month")) > 12:
            return render_template("adjustment.html")
```

Displays the content of the adjustment journal based on the month and year inputted
```
        month = request.form.get("month")
        month = "0" + month
        year = request.form.get("year")
        journal = db.execute(
            "SELECT Date, DA_NAME, CA_NAME, Debit, Credit FROM Credit_Account JOIN Debit_Account ON Credit_Account.AJ_ID=Debit_Account.AJ_ID JOIN Adjustment_Journal ON Credit_Account.AJ_ID=Adjustment_Journal.AJ_ID WHERE strftime('%m', Date) = ? AND strftime('%Y', Date) = ? ORDER BY Date",
            month,
            year,
        )

        return render_template("adjustment_journal.html", journal=journal)
```

Renders the adjustment.html
```
    if request.method == "GET":
        return render_template("adjustment.html")
```

Selects the revenue debit by summing up all the debits from the general and adjustment journal where the debit type is revenue
```
@app.route("/income", methods=["GET", "POST"])
def income_statement():
    if request.method == "POST":
        if 1 > int(request.form.get("month")) > 12:
            return render_template("income.html")
        month = request.form.get("month")
        month = "0" + month
        year = request.form.get("year")

        revenue_debit = db.execute(
            "SELECT SUM(Debit) FROM ( SELECT Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Revenue' AND strftime('%m', General_Journal.Date) = ? AND strftime('%Y', General_Journal.Date) = ? UNION ALL SELECT Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Revenue' AND strftime('%m', Adjustment_Journal.Date) = ? AND strftime('%Y', Adjustment_Journal.Date) = ?)",
            month,
            year,
            month,
            year,
        )
```

Selects the revenue credit by summing up all the credits from the general and adjustment journal where the credit type is revenue
```
        revenue_credit = db.execute(
            "SELECT SUM(Credit) FROM ( SELECT Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Revenue' AND strftime('%m', General_Journal.Date) = ? AND strftime('%Y', General_Journal.Date) = ? UNION ALL SELECT Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Revenue' AND strftime('%m', Adjustment_Journal.Date) = ? AND strftime('%Y', Adjustment_Journal.Date) = ?)",
            month,
            year,
            month,
            year,
        )
```

Selects the expense debit by summing up all the debits from the general and adjustment journal where the debit type is expense
```
        expense_debit = db.execute(
            "SELECT DA_Name, SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Expense' AND strftime('%m', General_Journal.Date) = ? AND strftime('%Y', General_Journal.Date) = ? UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Expense' AND strftime('%m', Adjustment_Journal.Date) = ? AND strftime('%Y', Adjustment_Journal.Date) = ?) GROUP BY DA_Name",
            month,
            year,
            month,
            year,
        )
```

Selects the total expense by summing up all the debits from the general and adjustment journal where the debit type is expense
```
        total_expense = db.execute(
            "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Expense' AND strftime('%m', General_Journal.Date) =  ?  AND strftime('%Y', General_Journal.Date) =  ?  UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Expense' AND strftime('%m', Adjustment_Journal.Date) =  ?  AND strftime('%Y', Adjustment_Journal.Date) =  ? )",
            month,
            year,
            month,
            year,
        )
```

Calculates the revenue by subtracting revenue debit from revenue credit
```
revenue_debit = revenue_debit[0]["SUM(Debit)"]
revenue_credit = revenue_credit[0]["SUM(Credit)"]
if revenue_debit == None:
    revenue_debit = 0
if revenue_credit == None:
    revenue_credit = 0
revenue = revenue_credit - revenue_debit
```

Renders the income.html
```
if request.method == "GET":
    return render_template("income.html")
```

Selects the revenue credit by summing up all the credits from the general and adjustment journal where the credit type is revenue
```
@app.route("/owner", methods=["GET", "POST"])
def owner_equity():
    if request.method == "POST":
            if 1 > int(request.form.get("month")) > 12:
                return render_template("owner.html")
            month = request.form.get("month")
            month = "0" + month
            year = request.form.get("year")

            date= year+"-"+month

            revenue_credit = db.execute(
            "SELECT SUM(Credit) FROM ( SELECT Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Revenue' AND strftime('%Y-%m', Date) < ? UNION ALL SELECT Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Revenue' AND strftime('%Y-%m', Date) < ?)",
            date,
            date
            )
```

Selects the total expense by summing up all the debits from the general and adjustment journal where the debit type is expense
```
            total_expense = db.execute(
            "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Expense' AND strftime('%Y-%m', Date) < ? UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Expense' AND strftime('%Y-%m', Date) < ?)",
            date,
            date
            )
```

Selects the capital credit by summing all the credits from the general and adjustment journal where the credit account is owner capital
```
            capital_credit = db.execute(
            "SELECT SUM(Credit) FROM ( SELECT CA_Name, Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE CA_Name = 'Owner Capital' AND strftime('%Y-%m', Date) < ? UNION ALL SELECT CA_Name, Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE CA_Name = 'Owner Capital' AND strftime('%Y-%m', Date) < ?)",
            date,
            date
        )
```

Selects the drawing debit by summing all the debits from the general and adjustment journal where the debit account is owner drawing
```
            drawing_debit = db.execute(
            "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE DA_Name = 'Owner Drawing' AND strftime('%Y-%m', Date) < ? UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE DA_Name = 'Owner Drawing' AND strftime('%Y-%m', Date) < ?)",
            date,
            date
        )
```

Selects the current capital credit by summing all the credits from the general and adjustment journal where the credit account is owner capital
```
            current_capital_credit = db.execute(
            "SELECT SUM(Credit) FROM ( SELECT CA_Name, Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE CA_Name = 'Owner Capital' AND strftime('%m', General_Journal.Date) = ? AND strftime('%Y', General_Journal.Date) = ? UNION ALL SELECT CA_Name, Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE CA_Name = 'Owner Capital' AND strftime('%m', Adjustment_Journal.Date) = ? AND strftime('%Y', Adjustment_Journal.Date) = ?)",
            month,
            year,
            month,
            year,
        )
```

Selects the current drawing debit by summing all the debits from the general and adjustment journal where the debit account is owner drawing
```
            current_drawing_debit = db.execute(
            "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE DA_Name = 'Owner Drawing' AND strftime('%m', General_Journal.Date) =  ?  AND strftime('%Y', General_Journal.Date) =  ?  UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE DA_Name = 'Owner Drawing' AND strftime('%m', Adjustment_Journal.Date) =  ?  AND strftime('%Y', Adjustment_Journal.Date) =  ? )",
            month,
            year,
            month,
            year,
        )
```

Calculates the revenue by summing the revenue and capital credit, then subtracting the total expense and drawing debit from it
```
revenue_credit = revenue_credit[0]["SUM(Credit)"]
if revenue_credit == None:
    revenue_credit = 0

total_expense = total_expense[0]["SUM(Debit)"]
if total_expense == None:
    total_expense = 0

capital_credit = capital_credit[0]["SUM(Credit)"]
if capital_credit == None:
    capital_credit = 0

drawing_debit = drawing_debit[0]["SUM(Debit)"]
if drawing_debit == None:
    drawing_debit = 0

current_capital_credit = current_capital_credit[0]["SUM(Credit)"]
if current_capital_credit == None:
    current_capital_credit = 0

current_drawing_debit = current_drawing_debit[0]["SUM(Debit)"]
if current_drawing_debit == None:
    current_drawing_debit = 0

owner_capital= revenue_credit + capital_credit - total_expense - drawing_debit
```

Selects the revenue debit 2 by summing up all the debits from the general and adjustment journal where the debit type is revenue
```
            revenue_debit2 = db.execute(
                "SELECT SUM(Debit) FROM ( SELECT Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Revenue' AND strftime('%m', General_Journal.Date) = ? AND strftime('%Y', General_Journal.Date) = ? UNION ALL SELECT Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Revenue' AND strftime('%m', Adjustment_Journal.Date) = ? AND strftime('%Y', Adjustment_Journal.Date) = ?)",
                month,
                year,
                month,
                year,
            )
```

Selects the revenue credit by summing up all the credits from the general and adjustment journal where the credit type is revenue
```
            revenue_credit2 = db.execute(
                "SELECT SUM(Credit) FROM ( SELECT Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Revenue' AND strftime('%m', General_Journal.Date) = ? AND strftime('%Y', General_Journal.Date) = ? UNION ALL SELECT Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Revenue' AND strftime('%m', Adjustment_Journal.Date) = ? AND strftime('%Y', Adjustment_Journal.Date) = ?)",
                month,
                year,
                month,
                year,
            )
```

Selects the total expense 2 by summing up all the debits from the general and adjustment journal where the debit type is expense
```
            total_expense2 = db.execute(
                "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Expense' AND strftime('%m', General_Journal.Date) =  ?  AND strftime('%Y', General_Journal.Date) =  ?  UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Expense' AND strftime('%m', Adjustment_Journal.Date) =  ?  AND strftime('%Y', Adjustment_Journal.Date) =  ? )",
                month,
                year,
                month,
                year,
            )
```

Calculates the income by subtracting the total expense 2 from revenue credit 2 and debit 2
```
revenue_debit2 = revenue_debit2[0]["SUM(Debit)"]
revenue_credit2 = revenue_credit2[0]["SUM(Credit)"]
if revenue_debit2 == None:
    revenue_debit2 = 0
if revenue_credit2 == None:
    revenue_credit2 = 0
revenue2 = revenue_credit2 - revenue_debit2

total_expense2 = total_expense2[0]["SUM(Debit)"]

income=revenue2-total_expense2

return render_template(
    "owner_equity.html",
    owner_capital=owner_capital,
    current_capital_credit=current_capital_credit,
    current_drawing_debit=current_drawing_debit,
    income=income
)
```

Renders the owner.html
```
    if request.method == "GET":
        return render_template("owner.html")
```

Selects the revenue credit by summing all the credits from the general and adjustment journal where the type is revenue
```
@app.route("/balance", methods=["GET", "POST"])
def balance_sheet():
    if request.method == "POST":
            if 1 > int(request.form.get("month")) > 12:
                return render_template("balance.html")
            month = request.form.get("month")
            month = "0" + month
            year = request.form.get("year")

            date= year+"-"+month

            revenue_credit = db.execute(
            "SELECT SUM(Credit) FROM ( SELECT Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Revenue' AND strftime('%Y-%m', Date) < ? UNION ALL SELECT Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Revenue' AND strftime('%Y-%m', Date) < ?)",
            date,
            date
            )
```

Selects the total expense by summing all the debits from the general and adjustment journal where the type is expense
```
            total_expense = db.execute(
            "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Expense' AND strftime('%Y-%m', Date) < ? UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Expense' AND strftime('%Y-%m', Date) < ?)",
            date,
            date
            )
```

Selects the capital credit by summing all the credits from the general and adjustment journal where the credit account is owner capital
```
            capital_credit = db.execute(
            "SELECT SUM(Credit) FROM ( SELECT CA_Name, Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE CA_Name = 'Owner Capital' AND strftime('%Y-%m', Date) < ? UNION ALL SELECT CA_Name, Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE CA_Name = 'Owner Capital' AND strftime('%Y-%m', Date) < ?)",
            date,
            date
        )
```

Selects the drawing debit by summing all the debits from the general and adjustment journal where the debit account is owner drawing
```
            drawing_debit = db.execute(
            "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE DA_Name = 'Owner Drawing' AND strftime('%Y-%m', Date) < ? UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE DA_Name = 'Owner Drawing' AND strftime('%Y-%m', Date) < ?)",
            date,
            date
        )
```

Selects the current capital credit by summing all the credits from the general and adjustment journal where the credit account is owner capital
```
            current_capital_credit = db.execute(
            "SELECT SUM(Credit) FROM ( SELECT CA_Name, Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE CA_Name = 'Owner Capital' AND strftime('%m', General_Journal.Date) = ? AND strftime('%Y', General_Journal.Date) = ? UNION ALL SELECT CA_Name, Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE CA_Name = 'Owner Capital' AND strftime('%m', Adjustment_Journal.Date) = ? AND strftime('%Y', Adjustment_Journal.Date) = ?)",
            month,
            year,
            month,
            year,
        )
```

Selects the current drawing debit by summing all the debits from the general and adjustment journal where the debit account is owner drawing
```
            current_drawing_debit = db.execute(
            "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE DA_Name = 'Owner Drawing' AND strftime('%m', General_Journal.Date) =  ?  AND strftime('%Y', General_Journal.Date) =  ?  UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE DA_Name = 'Owner Drawing' AND strftime('%m', Adjustment_Journal.Date) =  ?  AND strftime('%Y', Adjustment_Journal.Date) =  ? )",
            month,
            year,
            month,
            year,
        )
```

Calculates the owner capital by summing up the revenue and capital credit, then subtracting the total expense and drawing debit from it
```
revenue_credit = revenue_credit[0]["SUM(Credit)"]
if revenue_credit == None:
    revenue_credit = 0

total_expense = total_expense[0]["SUM(Debit)"]
if total_expense == None:
    total_expense = 0

capital_credit = capital_credit[0]["SUM(Credit)"]
if capital_credit == None:
    capital_credit = 0

drawing_debit = drawing_debit[0]["SUM(Debit)"]
if drawing_debit == None:
    drawing_debit = 0

current_capital_credit = current_capital_credit[0]["SUM(Credit)"]
if current_capital_credit == None:
    current_capital_credit = 0

current_drawing_debit = current_drawing_debit[0]["SUM(Debit)"]
if current_drawing_debit == None:
    current_drawing_debit = 0

owner_capital= revenue_credit + capital_credit - total_expense - drawing_debit
```

Selects the revenue debit 2 by summing all the debits from the general and adjustment journal where the type is revenue
```
            revenue_debit2 = db.execute(
                "SELECT SUM(Debit) FROM ( SELECT Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Revenue' AND strftime('%m', General_Journal.Date) = ? AND strftime('%Y', General_Journal.Date) = ? UNION ALL SELECT Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Revenue' AND strftime('%m', Adjustment_Journal.Date) = ? AND strftime('%Y', Adjustment_Journal.Date) = ?)",
                month,
                year,
                month,
                year,
            )
```

Selects the revenue credit 2 by summing all the credits from the general and adjustment journal where the type is revenue
```
            revenue_credit2 = db.execute(
                "SELECT SUM(Credit) FROM ( SELECT Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Revenue' AND strftime('%m', General_Journal.Date) = ? AND strftime('%Y', General_Journal.Date) = ? UNION ALL SELECT Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Revenue' AND strftime('%m', Adjustment_Journal.Date) = ? AND strftime('%Y', Adjustment_Journal.Date) = ?)",
                month,
                year,
                month,
                year,
            )
```

Selects the total expense 2 by summing all the debits from the general and adjustment journal where the type is expense
```
            total_expense2 = db.execute(
                "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Expense' AND strftime('%m', General_Journal.Date) =  ?  AND strftime('%Y', General_Journal.Date) =  ?  UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Expense' AND strftime('%m', Adjustment_Journal.Date) =  ?  AND strftime('%Y', Adjustment_Journal.Date) =  ? )",
                month,
                year,
                month,
                year,
            )
```

Calculates the  capital by summing up the owner capital and current capital credit, then subtracting the current drawing debit, revenue debit 2 and credit 2, and total expense 2 from it
```
revenue_debit2 = revenue_debit2[0]["SUM(Debit)"]
revenue_credit2 = revenue_credit2[0]["SUM(Credit)"]
if revenue_debit2 == None:
    revenue_debit2 = 0
if revenue_credit2 == None:
    revenue_credit2 = 0
revenue2 = revenue_credit2 - revenue_debit2

total_expense2 = total_expense2[0]["SUM(Debit)"]
if total_expense2 == None:
    total_expense2 = 0

income=revenue2-total_expense2

capital = owner_capital + current_capital_credit - current_drawing_debit + income
```

Selects the total asset debit by summing all the debits from the general and adjustment journal where the type is asset
```
            total_Asset_Debit = db.execute(
                "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Asset' AND strftime('%Y-%m', Date) <= ?  UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Asset' AND strftime('%Y-%m', Date) <= ? )",
            date,
            date
            )
```

Selects the total asset credit by summing all the credits from the general and adjustment journal where the type is asset
```
            total_Asset_Credit = db.execute(
                "SELECT SUM(Credit) FROM ( SELECT CA_Name, Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Asset' AND strftime('%Y-%m', Date) <= ?  UNION ALL SELECT CA_Name, Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Asset' AND strftime('%Y-%m', Date) <= ? )",
            date,
            date
            )
```

Selects the total asset depreciation credit by summing all the credits from the general and adjustment journal where the type is asset depreciation
```
            total_Asset_Depreciation_Credit = db.execute(
                "SELECT SUM(Credit) FROM ( SELECT CA_Name, Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Asset_Depreciation' AND strftime('%Y-%m', Date) <= ?  UNION ALL SELECT CA_Name, Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Asset_Depreciation' AND strftime('%Y-%m', Date) <= ? )",
            date,
            date
            )
```

Selects the total liabilities debit by summing all the debits from the general and adjustment journal where the type is liabilities
```
            total_Liabilities_Debit = db.execute(
                "SELECT SUM(Debit) FROM ( SELECT DA_Name, Debit FROM Debit_Account JOIN General_Journal ON Debit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Liabilities' AND strftime('%Y-%m', Date) <= ?  UNION ALL SELECT DA_Name, Debit FROM Debit_Account JOIN Adjustment_Journal ON Debit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Liabilities' AND strftime('%Y-%m', Date) <= ? )",
            date,
            date
            )
```

Selects the total liabilities credit by summing all the credits from the general and adjustment journal where the type is liabilities
```
            total_Liabilities_Credit = db.execute(
                "SELECT SUM(Credit) FROM ( SELECT CA_Name, Credit FROM Credit_Account JOIN General_Journal ON Credit_Account.GJ_ID = General_Journal.GJ_ID WHERE Type = 'Liabilities' AND strftime('%Y-%m', Date) <= ?  UNION ALL SELECT CA_Name, Credit FROM Credit_Account JOIN Adjustment_Journal ON Credit_Account.AJ_ID = Adjustment_Journal.AJ_ID WHERE Type = 'Liabilities' AND strftime('%Y-%m', Date) <= ? )",
            date,
            date
            )
```

Displays the total liabilities debit and credit and total asset debit and credit
```
            total_Asset_Debit = total_Asset_Debit[0]["SUM(Debit)"]
            total_Asset_Credit = total_Asset_Credit[0]["SUM(Credit)"]
            if total_Asset_Debit == None:
                total_Asset_Debit = 0
            if total_Asset_Credit == None:
                total_Asset_Credit = 0

            total_Liabilities_Debit = total_Liabilities_Debit[0]["SUM(Debit)"]
            total_Liabilities_Credit = total_Liabilities_Credit[0]["SUM(Credit)"]
            if total_Liabilities_Debit == None:
                total_Liabilities_Debit = 0
            if total_Liabilities_Credit == None:
                total_Liabilities_Credit = 0
            print(total_Liabilities_Debit)
            print(total_Liabilities_Credit)
            print(total_Asset_Debit)
            print(total_Asset_Credit)
            total_Asset_Depreciation_Credit = total_Asset_Depreciation_Credit[0]["SUM(Credit)"]
            if total_Asset_Depreciation_Credit == None:
                total_Asset_Depreciation_Credit = 0

            total_Asset = total_Asset_Debit - total_Asset_Credit
            total_Liabilities = total_Liabilities_Credit - total_Liabilities_Debit
            total_Asset_Depreciation = total_Asset_Depreciation_Credit



            return render_template(
                "balance_sheet.html",
                capital=capital,
                total_Asset = total_Asset,
                total_Liabilities= total_Liabilities,
                total_Asset_Depreciation = total_Asset_Depreciation
            )
```

Renders the balance.html
```
    if request.method == "GET":
        return render_template("balance.html")
```
