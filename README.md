## 优雅的写PHP代码
#### 对象和数据结构
##### 使用 `getters` 和 `setters`
##### PHP 里面你可以给方法设置 `public`, `protected` 和  `private` 关键词. 通过它们你可以控制对属性的修改。
* 当你想要做获取一个对象属性开外的事情，你无需查找和改变代码库里的每个访问点
* 做 `set`. 操作时使添加验证变得简单
* 将内部表述包裹起来
* 易于添加日志和错误处理
* 继承该类，你就可以重写默认的方法
* 你可以对对象属性懒惰加载，比如加载属性是来自服务器.
#### 这样才符合`开闭原则`
* 错误示例
```
class BankAccount{        public $balance = 1000;}$bankAccount = new BankAccount();// 购买鞋子 ...$bankAccount->balance -= 100;
```
* 正确示例
```
class BankAccount{        private $balance;        public function __construct($balance = 1000)    {            $this->balance = $balance;        }        public function withdrawBalance($amount)    {                if ($amount > $this->balance) {                        throw new \Exception('Amount greater than available balance.');                }                        $this->balance -= $amount;        }            public function depositBalance($amount)    {                $this->balance += $amount;        }            public function getBalance()    {                return $this->balance;        } }$bankAccount = new BankAccount();// 购买鞋子...$bankAccount->withdrawBalance($shoesPrice);// 获取余额$balance = $bankAccount->getBalance();
```
#### 让对象具有 private/protected 成员
* 错误示例
```
class Employee{        public $name;        public function __construct($name)    {                $this->name = $name;        }}$employee = new Employee('John Doe');echo 'Employee name: '.$employee->name; // Employee name: John Doe
```
* 正确示例
```
class Employee{        private $name;            public function __construct($name)    {                $this->name = $name;        }                public function getName() {                return $this->name;        }}$employee = new Employee('John Doe');echo 'Employee name: '.$employee->getName(); // Employee name: John Doe
```
#### 接口隔离原则 (ISP)
###### ISP 描述了 “客户端应该不被强迫依赖于它们用不到的接口。”
一个演示这条原则的绝佳例子是那些要求大量设置对象的类。不要要求客户端设置一大串选项，这会带来收益，因为大多数场景它们都不需要用到那么多东西。使它们变得可选，防止出现“胖接口”。
* 错误示例
```
interface Employee{        public function work();        public function eat();}class Human implements Employee{        public function work()    {                // ....working        }            public function eat()    {                // ...... eating in lunch break        }}class Robot implements Employee{        public function work()    {                //.... working much more        }            public function eat()    {                //.... robot can't eating, but it must implement this method        }}
```
* 正确示例
###### 不是所有 workder 都是 employee，但每个 employee 都是 worker。
```
interface Workable{        public function work();}interface Feedable{        public function eat();}interface Employee extends Feedable, Workable{}class Human implements Employee{        public function work()    {                // ....working        }            public function eat()    {                //.... eating in lunch break        }}// robot can only workclass Robot implements Workable{        public function work()    {                // ....working        }}
```
#### 依赖倒置原则 (DIP)
##### 这条原则表述了两个基本事情
1. 高级别模块不应该依赖于低层级模块，两者都应依赖于抽象单元
2. 抽象单元不应依赖于实体，实体应该依赖于抽象单元

##### 一开始这可能比较难理解，但如果你接触过 PHP 框架（比如 Symfony），你就会在依赖注入（DI）的实现上看到这一原则的运用。尽管它们不是理想的概念，DIP 保证了高级别的模块无需知道低层模块的实现细节，就能对它们进行设置，这可以通过 DI 来完成。这带来的巨大收益是它减少了模块之间的耦合。耦合性是非常糟糕的开发模式，因为这使你的代码难以重构。
* 错误示例
```
class Employee{    public function work()  {        // ....working    }}class Robot extends Employee{        public function work()    {                //.... working much more        }}class Manager{        private $employee;        public function __construct(Employee $employee)    {                $this->employee = $employee;        }            public function manage()    {                $this->employee->work();        }}
```
* 正确示例
```
interface Employee{        public function work();}class Human implements Employee{        public function work()    {                // ....working        }}class Robot implements Employee{        public function work()    {                //.... working much more        }}class Manager{        private $employee;        public function __construct(Employee $employee)    {                $this->employee = $employee;        }            public function manage()    {                $this->employee->work();        }}
```
#### 使用方法链
##### 这个模式很有用，且许多类库都用到这个模式，比如 PHPUnit 和 Doctrine。它允许你的代码具有表达性，减少冗余。为此，使用方法链，对比看看代码如何变得整洁。在你的类里面的函数中，在每一个 set 函数后面简单使用 return $this ，就能实现了。
* 错误示例
```
class Car {        private $make, $model, $color;            public function __construct() {                $this->make = 'Honda';                $this->model = 'Accord';                $this->color = 'white';        }                public function setMake($make) {                $this->make = $make;        }                public function setModel($model) {                $this->model = $model;        }                public function setColor($color) {                $this->color = $color;        }                public function dump() {                var_dump($this->make, $this->model, $this->color);        }}$car = new Car();$car->setColor('pink');$car->setMake('Ford');$car->setModel('F-150');$car->dump();
```
* 正确示例
```
class Car {        private $make, $model, $color;            public function __construct() {                $this->make = 'Honda';                $this->model = 'Accord';                $this->color = 'white';        }                public function setMake($make) {                $this->make = $make;                        // NOTE: Returning this for chaining                return $this;        }                public function setModel($model) {                $this->model = $model;                        // NOTE: Returning this for chaining                return $this;        }                public function setColor($color) {                $this->color = $color;                        // NOTE: Returning this for chaining                return $this;        }                public function dump() {                var_dump($this->make, $this->model, $this->color);        }}$car = (new Car())  ->setColor('pink')  ->setMake('Ford')  ->setModel('F-150')  ->dump();
```
#### 组合优于继承
##### 正如四人帮著名的 《设计模式》所言，你应该尽可能使用组合而非继承。不管是使用继承还是组合都有很多好的理由。主要的点是，如果你的本能想法是想通过继承来实现，那么试着想想如果用组合的方式来实现，你的问题是否能更好地解决。一些情况下会的。
##### 你可能会问，“我什么时候该用继承？” 这取决于你手头遇到的问题，但以下是使用继承优于组合的场景列表：
##### 你的继承表达了 “is - a” 的关系，且不是 “has - a” 的关系 （人类 -> 动物 vs. 用户 -> 用户详情).
##### 你可以重复使用来自基类的代码（像所有动物一样，人类可以 move）
##### 你想通过改变基类的方式对下游类作出全局改变（所有动物在 move 的时候都会改变热量消耗）
* 错误示例
```
class Employee {        private $name, $email;            public function __construct($name, $email) {                $this->name = $name;                $this->email = $email;        }            // ...}// Bad because Employees "have" tax data. // EmployeeTaxData is not a type of Employeeclass EmployeeTaxData extends Employee {        private $ssn, $salary;            public function __construct($name, $email, $ssn, $salary) {        parent::__construct($name, $email);                $this->ssn = $ssn;                $this->salary = $salary;        }            // ...}
```
* 正确示例
```
class EmployeeTaxData {        private $ssn, $salary;            public function __construct($ssn, $salary) {                $this->ssn = $ssn;                $this->salary = $salary;        }            // ...}class Employee {        private $name, $email, $taxData;            public function __construct($name, $email) {                $this->name = $name;                $this->email = $email;        }                public function setTaxData($ssn, $salary) {                $this->taxData = new EmployeeTaxData($ssn, $salary);        }        // ...}
```
#### 不重复原则 (DRY)
##### 尝试观察 DRY 原则。
##### 要尽全力避免重复代码，重复代码之所以糟糕是因为这意味着如果你要对一些逻辑作出改变，你就需要修改不止一个地方。
##### 想象你在经营一家餐厅，你要对库存进行跟踪：所有番茄、洋葱、大蒜、辣椒等等。如果你有多个列表，一直下去当你递送一碟番茄的时候你就不得不要更新所有的列表。如果你只有一个列表，那就只需要更新一个地方！
##### 重复代码的出现经常是因为你有两个或更多轻微的不同，但是大部分逻辑又可以共同，而他们的差异强迫你通过两个或更多分开的函数来处理许多相同的事情。移除重复代码意味着创建一个可以处理这一系列不同事情的唯一抽象单元（如一个函数/模块/类）。
##### 正确获取抽象单元至关重要，所以你需要遵循 SQL-ID 原则。糟糕的抽象单元要比重复代码更加差，所以小心为妙！要是你能做一个好的抽象单元，那就做吧！不要写重复代码，不然你想要改变一个东西的时候，你会发现你需要改变好几个地方。
* 错误示例
```
function showDeveloperList($developers){     foreach ($developers as $developer) {                $expectedSalary = $developer->calculateExpectedSalary();           $experience = $developer->getExperience();                $githubLink = $developer->getGithubLink();                $data = [                        $expectedSalary,                        $experience,                        $githubLink        ];                        render($data);        }}function showManagerList($managers){     foreach ($managers as $manager) {                $expectedSalary = $manager->calculateExpectedSalary();             $experience = $manager->getExperience();                $githubLink = $manager->getGithubLink();                $data = [                        $expectedSalary,                        $experience,                        $githubLink        ];                        render($data);        }}
```
* 正确示例
```
function showList($employees){     foreach ($employees as $employee) {             $expectedSalary = $employee->calculateExpectedSalary();            $experience = $employee->getExperience();            $githubLink = $employee->getGithubLink();                $data = [                        $expectedSalary,                        $experience,                        $githubLink        ];                        render($data);        }}
```
##### 非常优秀的:
##### 更好的是使用代码的紧凑版 .
```
function showList($employees){     foreach ($employees as $employee) {                render([                        $employee->calculateExpectedSalary(),                        $employee->getExperience(),                        $employee->getGithubLink()                ]);        }}
```


