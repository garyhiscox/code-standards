ColdFusion Best Practices
=========================

**Author:** Adam Presley

1. Scoping
----------
Scoping in ColdFusion defines the context in which variables are valid. There
are a few rules and gotcha's for scoping in ColdFusion which will be outlined
here.

* *All* variables in components (CFC) must be prefixed with a scope
   * See **Section 2.1** for more information on function scoping in components
* Any variable in a CFM file must be prefixed with a scope, *except* those
   intended for the **variables** scope.
   - This is the default scope for CFM pages
   - Not prefixing here does no harm

In this example we'll see a CFC and CFM page using scopes correctly.

**Example CFC**
```js
component {
   this.OverTimeMultiplier = 1.5;
   this.HoursPerWeek = 40;

   public PayService function init() {
      variables.OverTimeCalculatorService = new OverTimeCalculatorService(
         HoursPerWeek=this.HoursPerWeek
      );

      return this;
   }

   public numeric function calculateWeeklyPay(
         required Employee EmployeeRecord,
         required numeric HoursWorked
   ) {
      var OverTimeHours = variables.OvertimeCalculatorService.determineOTHours(
         HoursWorked=arguments.HoursWorked
      );

      var RegularHours = arguments.HoursWorked - OverTimeHours;

      var RegularPay = calculateRegularPay(
         EmployeeRecord=arguments.EmployeeRecord,
         Hours=RegularHours
      );

      var OverTimePay = calculateOverTimePay(
         EmployeeRecord=arguments.EmployeeRecord,
         Hours=OverTimeHours
      );

      return RegularPay + OverTimePay;
   }

   public numeric function calculateRegularPay(
         required Employee EmployeeRecord,
         required numeric Hours
   ) {
      return arguments.Hours * arguments.EmployeeRecord.HourlySalary;
   }

   public numeric function calculateOverTimePay(
         required Employee EmployeeRecord,
         required numeric Hours
   ) {
      return arguments.hours * (
         arguments.EmployeeRecord.HourlySalary * this.OverTimeMultiplier
      );
   }
}
```

**Example CFM**
```cfm
<cfset request.ParagraphStyle = request.Styles.getParagraphStyle() />

<cfoutput>

<p class="#request.ParagraphStyle#">
   Hello #Session.Employee.Name#.<br />
   Your pay this week is #request.rc.EmployeePay#
</p>

</cfoutput>
```

2. Functions
------------

### 2.1 Scoping in Functions
* Use the var keywork to scope variables local to the function
* Local variables *must be* declared using the form ```var MyVariable```

### 2.2 Naming
Functions perform actions. As such they should make use of verbs. Furthermore
most functions tend to operate on something, and as such should clearly call
out what they are doing and to what they are performing this action on.

**Example**
```js
public array function splitSentenceIntoWords(required string Sentence) {
   return arguments.Sentence.split(" ");
}
```

Notice how the verb is **split**, and we can easily see we are acting on a
sentence. That is the noun.

There are times when a simple verb may suffice. If the function is part of a
component where the intent or domain is clear, then the function name could
be shortened to reflect only what it is doing. However be cautious.
More clarity is better than assuming everyone reading your code understands
the domain.

**Example: MailProcessor.cfc**
```js
component {
   public void function process(required numeric QueueId) {
      // ...
   }
}
```

### 2.3 Output
A function should never directly send output to the browser. This is
the responsibility of a view page or rendering component. Exceptions to this would
be if the function is part of a rendering component in a framework or system
that outputs contents to the browser as part of the request process.

**Example**
```js
public void function doSomething() {
   // ...
}
```

### 2.4 Calling Functions
When calling user-defined functions in ColdFusion it is prefered to use *Named Arguments*.

**Example of Named Arguments**
```js
public numeric function ordered(
  numeric Arg1,
  numeric Arg2) {
   //..
}

result = named(
  Arg1=1,
  Arg2=2
);
```

**Example of Ordered Arguments**
```js
public numeric function ordered(
  numeric Arg1,
  numeric Arg2
) {
   //..
}

result = ordered(1, 2);
```

In most situations using *Named Arguments* is prefered as it offers additional
clarity by allowing the reader to understand what each argument is. There are
some times when using *Named Arguments* might be a bit over the top and wordy.
As such use your best judgement when calling functions. Ask yourself, "Can the
reader of this code read it like a sentence? Is the intent clear?" Here is
an example of where *Ordered Arguments* might be clearer.

```js
public numeric function sum(numeric leftOperand, numeric rightOperand) {
   return arguments.leftOperand + arguments.rightOperand;
}

result = sum(1, 2);
```
