#item class contains name, price, stock of the items
class Item:
    def __init__(self,item_name,item_price,item_stock):
        self.price=item_price
        self.name=item_name
        self.stock=item_stock
        
    # Function to update the stock of an item 
    def stockUpdate(self,stock):
        self.stock=stock

    # decrement item from stock when a customer buys it
    def buy(self):
        if self.stock==0:
            pass
        self.stock-=1
       
# Coin class that  contains the value and stock of coin   
class Coin:
    def __init__(self,coin_value,coin_stock):
        self.value=coin_value
        self.stock=coin_stock

# exception class, if a customer inserts an invalid coin.
# it will give a message to customer to insert the valid coin 
class InvalidCoin(ZeroDivisionError):
    def __init__(self,arg):
        self.msg=arg

# class that  performs backened operation like transactions
class VendingMachine :
    # Constructor
    def __init__(self):
        # amount that a customer inserted
        self.amount=0
        #list of items that vending machine have
        self.items=[]
        #list of coind denominations, vending machine have
        self.coins=[]
    # Function to add new Coin denomination into vending machine
    def addCoin(self,coin):
        self.coins.append(coin)
        
    # Function to add new item into vending machine
    def addItem(self,item):
        self.items.append(item)
        
    # Function to show menu to the customer
    def showItems(self):
        print('Items available:\n')
        print('  ','Name',' Price' )
        i=1
        for item in self.items:
            #if that item is in stock
            if item.stock>0:
                print(str(i)+'.',item.name,'  ',item.price,sep=' ')
                i+=1
                
    def addCash(self,amount):
        self.amount+=amount

    # Function to check if a customer can buy this item or not.
    # if he can buy then decrement from stock
    def buyItem(self,item):
        #Condition to check  has he inserted enough amount to buy this item
        if self.amount<item.price:
            # he has not inserted enough amount print this message
            print("Insufficient money to buy this item, insert more coins.")
        # he has inserted enough amount so else block will execute
        else:
            # Condition to check that vending machine have change money to return the
            #customer or not
            if self.transactionPossible(self.amount-item.price):
                self.amount-=item.price
                item.buy()
                print('You got '+item.name)
                print('Cash Remaining: '+str(self.amount),'\n')
            # if vending machine don't have change then we will cancel this transaction
            # and refund his money
            else:
                print("Sorry i don,t have change money ")
                self.checkRefund()

    # Function to check that Vending machine have that item or not
    def containsItem(self,requested_item):
        for item in self.items:
            if item.stock>0 and item.name==requested_item:
                return True
        return False
    
    # Function to check that coin inserted by customer machine accepts or not
    def containsCoin(self,inserted_coin):
        for coin in self.coins:
            if inserted_coin==coin.value:
                return True
        return False

    # Function to pick the item inserted by customer
    def getItem(self,requested_item):
        ret=None
        for item in self.items:
            if item.name==requested_item:
                ret=item
                break
        return ret

    # Function to Saccept cash inserted by customer to buy item
    def insertAmount(self,item):
        price=item.price
        print('Press 0 to cancel the transaction ')
        while self.amount<price:
            try:
                money=float(input('insert ' + str(price - self.amount) + ': '))
                # Condition to check if customer wnat to cancel this transaction
                if money==0.0:
                    print("Your last transaction is cancelled")
                    self.checkRefund()
                    return -1
                # condition to check that inserted coin is valid
                if self.containsCoin(money):
                    self.amount+=money
                    for i in self.coins:
                        if i.value==money:
                            i.stock+=1
                            break
                # if inserted coin is invalid
                # raise an exception
                else:
                    raise InvalidCoin("\nInvalid Coin, Please Insert a valid coin of Value 1,5,10,25")
            # exception handling
            except InvalidCoin as i:
                print(i.msg)
                print('Press 0 to cancel the transaction')

     # Function to check that if this transaction is possible or not
     # it will check that vending machine have the enough coin to refund the change money
     # it takes amount as an argument named 'refund_money' that vending machine need to refund
     # if machine have the change coins to refund it will return True else will return False
    def transactionPossible(self,refund_money):
        # using Dynamic Programming to solve this problem
        dp=[[-1 for i in range(int(refund_money+1))] for j in range(5)]
        # initialize dp for zero refund_money
        for i in range(5):
            dp[i][0]=0
        # since we have only four coins so iam using 5 for 4 coins
        # coins are in sorted order in coins list
        for i in range(1,5):
            for j in range(1,int(refund_money+1)):
                # check if coin stock is available or not
                if self.coins[i-1].stock>0 and j>=self.coins[i-1].value:
                    coins_needed=j//self.coins[i-1].value
                    if coins_needed<=self.coins[i-1].stock and dp[i][int(j-self.coins[i-1].value*coins_needed)]!=-1:
                        dp[i][j]=coins_needed+dp[i][int(j-self.coins[i-1].value*coins_needed)]
                    elif  j-self.coins[i-1].value*self.coins[i-1].stock>=0 and dp[i-1][int(j-self.coins[i-1].value*self.coins[i-1].stock)]!=-1:
                        dp[i][j]=self.coins[i-1].stock+dp[i-1][int(j-self.coins[i-1].value*self.coins[i-1].stock)]
                    else:
                        dp[i][j]=dp[i-1][j]
                # coin stock is zero
                else:
                    dp[i][j]=dp[i-1][j]
                    
        # code to extract the exact the cooin denominations that are required to refund the refunnd money
        # decrement those coin from coin stock
        i=4
        j=int(refund_money)
        if dp[4][int(refund_money)]!=-1:
            while i>0 and j>0:
                if dp[i][j]!=dp[i-1][j]:
                    coins_needed=j//self.coins[i-1].value
                    if coins_needed<=self.coins[i-1].stock:
                        self.coins[i-1].stock-=coins_needed
                        j=int(j-self.coins[i-1].value*coins_needed)
                    else:
                        self.coins[i-1].stock=0
                        j=int(j-self.coins[i-1].value*self.coins[i-1].stock)
                    i-=1
                        
                else:
                    i-=1
            return True
        return False
            
    # function to check if customer has any refund amount
    # if there is refund amount so refund that amount to customer
    def checkRefund(self):
        if self.amount>0 and self.transactionPossible(self.amount):
            print(str(self.amount)+" refunded.\n")
            self.amount=0
        print('Thank You, Have a nice day!\n')

# Function to start the machine
def vend():
    machine=VendingMachine()
    # Initializing the machine with three items
    # initiallize each item stock is 10
    coke=Item('coke',25,10)
    pepsi=Item('pepsi',35,10)
    soda=Item('soda',45,10)
    # Initializing coin denominations for machine
    # initially each coin have 10 stock
    penny=Coin(1.0,10)
    nickel=Coin(5.0,10)
    dime=Coin(10.0,10)
    quarter=Coin(25.0,10)
    # add items into machine
    machine.addItem(coke)
    machine.addItem(pepsi)
    machine.addItem(soda)
    # Add coins into machine
    machine.addCoin(penny)
    machine.addCoin(nickel)
    machine.addCoin(dime)
    machine.addCoin(quarter)
    print('Welcome to the Vending machine!\n')
    continueToBuy=True
    while continueToBuy==True:
        machine.showItems()
        selected=input('\nEnter item name to select: ')
        # check that the item selected by customer
        # machine contains or not
        if machine.containsItem(selected):
            item=machine.getItem(selected)
            if machine.insertAmount(item)!=-1:
                machine.buyItem(item)
            a=input('Want to buy Something else? (y/n): \n')
            # Condition to check if customer wants to buy something else or not
            if a=='n':
                continueToBuy=False
                machine.checkRefund()
            else:
                continue
        # if machine dont have the item selected by customer
        else:
            print('Item not available. Select another item.\n ')
            continue
# start the vending machine        
vend()        
        

