import time
from datetime import datetime as dt

# === Data Structures ===
accounts = {}
balances = {}
transactions = {}
limits = {}
request_queue = []

PANIC_PIN = "9999"
account_limits = {"saving": 100000, "current": 200000, "student": 50000}

# === Functions ===

def banner():
    print("\n" + "💳 VIRTUAL ATM 💳".center(40, "="))
    print("📅", dt.now().strftime("%A, %d %B %Y"), "\n")

def create_account(acc_no, pin):
    print("1. Saving\n2. Current\n3. Student")
    choice = input("Choose account type (1/2/3): ").strip()
    acc_type = {'1': 'saving', '2': 'current', '3': 'student'}.get(choice, 'saving')
    accounts[acc_no] = pin
    balances[acc_no] = 1000
    transactions[acc_no] = []
    limits[acc_no] = account_limits[acc_type]
    print(f"✅ {acc_type.title()} account created.\n")

def login():
    acc_no = input("👤 Account Number: ")
    pin = input("🔑 PIN: ")
    if pin == PANIC_PIN:
        print("🚨 SOMETHING WENT WRONG! Try later.\n")
        time.sleep(2)
        return None
    if acc_no in accounts and accounts[acc_no] == pin:
        return acc_no
    elif acc_no in accounts:
        for _ in range(2):
            pin = input("❌ Wrong PIN. Try again: ")
            if pin == PANIC_PIN:
                print("🚨 SOMETHING WENT WRONG! Try later.\n")
                time.sleep(2)
                return None
            if accounts[acc_no] == pin:
                return acc_no
        print("⛔ Too many attempts.\n")
        return None
    else:
        print("🆕 New user. Creating account...")
        create_account(acc_no, pin)
        return acc_no

def add_transaction(acc_no, t_type, amt):
    time_now = dt.now().strftime("%Y-%m-%d %H:%M:%S")
    transactions[acc_no].append((t_type, amt, time_now))
    if len(transactions[acc_no]) > 5:
        transactions[acc_no].pop(0)

def mini_statement(acc_no):
    print("\n🧾 Mini Statement:")
    for t in transactions[acc_no]:
        print(f"{t[2]} - {t[0]}: ₹{t[1]}")
    if not transactions[acc_no]:
        print("No transactions.")

def transaction_menu(acc_no):
    print(f"\n💰 Balance: ₹{balances[acc_no]}")
    print("1. Deposit\n2. Withdraw\n3. Check Balance\n4. Mini Statement")
    choice = input("Choose: ")

    if choice == '1':
        amt = float(input("Enter amount to deposit: ₹"))
        if amt > 0:
            balances[acc_no] += amt
            add_transaction(acc_no, "Deposit", amt)
            print("✅ Deposited.")
    elif choice == '2':
        amt = float(input("Enter amount to withdraw: ₹"))
        if 0 < amt <= balances[acc_no] and amt <= limits[acc_no]:
            balances[acc_no] -= amt
            add_transaction(acc_no, "Withdraw", amt)
            print("✅ Withdrawn.")
        else:
            print("❌ Invalid or exceeds balance/limit.")
    elif choice == '3':
        print(f"💰 Balance: ₹{balances[acc_no]}")
    elif choice == '4':
        mini_statement(acc_no)
    else:
        print("❌ Invalid choice.")

    time.sleep(1)

def run_atm():
    while True:
        banner()
        request_queue.append("ATM Request")
        acc = login()
        if acc:
            transaction_menu(acc)
        request_queue.pop(0)
        print("🔁 Returning to login...\n")
        time.sleep(1)

# === Start ATM ===
run_atm()
