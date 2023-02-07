## AOP：@AfterThrowing Advice Type
>參考網址：https://blog.csdn.net/owen_william/article/details/50812780
>
>使用@AfterThrowing註解可以修飾AfterThrowing增強處理，AfterThrowing增強處理主要用於處理程序中未處理的異常。使用@AfterThrowing註解時可指定如下的常用屬性：
>
>1) pointcut/value:這兩個屬性的作用是一樣的，它們都用於指定該切入點對應的切入表達式。一樣既可是一個已有的切入點，也可以直接定義切入點表達式。當指定了pointcut屬性後，value屬性值將會被覆蓋。
>
>2) throwing:該屬性指定一個形參名，用於表示Advice方法中可定義與此同名的形參，該形參可用於訪問目標方法拋出的異常。除此之外，在Advice方法中定義該參數時，指定的類型，會限制方法必須拋出指定類型的異常。

**稍微改寫一下DAO中的findAccounts方法，多一個tripWire布林參數，以便模擬例外錯誤狀況發生，當此值為TRUE則故意丟出一個RuntimeException例外**
```
@Component
public class AccountDAO {

	private String name;
	private String serviceCode;
	
	// add a new method: findAccounts()
	public List<Account> findAccounts(boolean tripWire){
		
		// for academic purpose ... simulate an exception
		if (tripWire) {
			throw new RuntimeException("No soup for you!!!");
		}
		
		List<Account> myAccounts = new ArrayList<>();
		
		// create sample accounts
		Account temp1 = new Account("John", "Silver");
		Account temp2 = new Account("Madhu", "Platinum");
		Account temp3 = new Account("Luca", "Gold");
		
		// add them to our accounts list
		myAccounts.add(temp1);
		myAccounts.add(temp2);
		myAccounts.add(temp3);	
		
		return myAccounts;
	}
}
```
**添加@AfterThrowing建議，此處的throwing參數，該參數可用於訪問目標方法拋出的異常；在此方法中，我們訪問例外錯誤並加以修改再重新拋回主應用程序**
```
@AfterThrowing(
	pointcut="execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))",
	throwing="theExc")
	public void afterThrowingFindAccountingAccountAdvice(
					JoinPoint theJoinPoint, Throwable theExc) {
		
		// print out which method we are advising on
		String method = theJoinPoint.getSignature().toShortString();
		System.out.println("\n=====>>> Excuting @AfterThrowing on method: " + method);
		
		// log the exception
		System.out.println("\n=====>>> The exception is: " + theExc);
	}
```
**建立AfterThrowingDemoApp演示應用主程序，在方法執行時故意帶入TRUE數值，使其方法執行時強制丟出例外錯誤，並由我們創建的Aspect攔截錯誤，改寫訊息後再重新丟出錯誤**
```
public class AfterThrowingDemoApp {

	public static void main(String[] args) {
		
		// read spring config java class
		AnnotationConfigApplicationContext context = 
				new AnnotationConfigApplicationContext(DemoConfig.class);
		
		// get the bean from spring container
		AccountDAO theAccountDAO = context.getBean("accountDAO", AccountDAO.class);
		
		// call method to find the accounts
		List<Account> theAccounts = null;
		
		try {
			// add a boolean flag to simulate exceptions
			boolean tripWire = true;
			theAccounts = theAccountDAO.findAccounts(tripWire);
		} 
		catch (Exception exc) {
			System.out.println("\n\nMain Program ... caught exception: " + exc);
		}

		// diaplay the accounts
		System.out.println("\n\nMain Program: AfterThrowingDemoApp");
		System.out.println("----");
		
		System.out.println(theAccounts);
		
		System.out.println("\n");		
		
		// close the context
		context.close();
	}
}
```
![image](https://user-images.githubusercontent.com/101872264/217273584-99c03114-97d8-47bb-b17e-38d4769042b4.png)


>## 結論：
> AOP的AfterThrowing處理雖然可以對目標方法的異常進行處理，但這種處理與直接使用catch捕捉不同，catch捕捉意味著完全處理該異常，如果catch塊中沒有重新拋出新的異常，則該方法可能正常結束；而AfterThrowing處理雖然處理了該異常，但它不能完全處理異常，該異常依然會傳播到上一級調用者，即JVM。
