# C++ Design Patterns

## 单例模式（Singleton Pattern）

### 概述
单例模式是一种创建型设计模式，确保一个类只有一个实例，并提供一个全局访问点。单例模式在全局范围内控制某个类的实例化，使得在整个应用程序中，只存在一个类的实例。

### 什么时候使用单例模式？
- 需要控制实例数量，确保一个类只有一个实例时。
- 需要一个全局访问点来访问某个类的唯一实例时。
- 当一个对象需要被多个部分共享时，如配置文件读取类、日志记录类等。

### 单例模式的实现
单例模式的实现有多种方法，包括饿汉式、懒汉式、线程安全的单例模式等。下面分别介绍这些实现方法，并说明其优缺点。

#### 饿汉式单例模式
饿汉式单例模式在类加载时就创建实例，线程安全，但在类未被使用时会占用内存。

```cpp
#include <iostream>

class Singleton {
private:
    static Singleton* instance;

    // 私有构造函数，防止外部实例化
    Singleton() {}

public:
    // 获取实例的方法
    static Singleton* getInstance() {
        return instance;
    }

    void showMessage() {
        std::cout << "Hello from Singleton!" << std::endl;
    }
};

// 在类加载时创建实例
Singleton* Singleton::instance = new Singleton();

int main() {
    Singleton* s = Singleton::getInstance();
    s->showMessage();
    return 0;
}
```
#### 懒汉式单例模式
懒汉式单例模式在第一次调用getInstance时创建实例，节省内存，但需要处理线程安全问题。
```cpp
#include <iostream>

class Singleton {
private:
    static Singleton* instance;

    // 私有构造函数，防止外部实例化
    Singleton() {}

public:
    // 获取实例的方法
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }

    void showMessage() {
        std::cout << "Hello from Singleton!" << std::endl;
    }
};

// 初始化静态成员变量
Singleton* Singleton::instance = nullptr;

int main() {
    Singleton* s = Singleton::getInstance();
    s->showMessage();
    return 0;
}
```
#### 线程安全的单例模式
为了确保线程安全，可以使用双重检查锁定（Double-Checked Locking）机制。
```cpp
#include <iostream>
#include <mutex>

class Singleton {
private:
    static Singleton* instance;
    static std::mutex mtx;

    // 私有构造函数，防止外部实例化
    Singleton() {}

public:
    // 获取实例的方法
    static Singleton* getInstance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);
            if (instance == nullptr) {
                instance = new Singleton();
            }
        }
        return instance;
    }

    void showMessage() {
        std::cout << "Hello from Singleton!" << std::endl;
    }
};

// 初始化静态成员变量
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;

int main() {
    Singleton* s = Singleton::getInstance();
    s->showMessage();
    return 0;
}
```
### 实际开发中的单例模式
#### 配置文件管理类
配置文件管理类是单例模式的典型应用之一，保证应用程序全局只存在一个配置文件管理实例。
```cpp
#include <iostream>
#include <fstream>
#include <map>
#include <string>
#include <mutex>

class ConfigManager {
private:
    static ConfigManager* instance;
    static std::mutex mtx;
    std::map<std::string, std::string> config;

    // 私有构造函数，读取配置文件
    ConfigManager() {
        std::ifstream file("config.txt");
        std::string key, value;
        while (file >> key >> value) {
            config[key] = value;
        }
        file.close();
    }

public:
    // 获取实例的方法
    static ConfigManager* getInstance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);
            if (instance == nullptr) {
                instance = new ConfigManager();
            }
        }
        return instance;
    }

    // 获取配置值
    std::string getValue(const std::string& key) {
        return config[key];
    }
};

// 初始化静态成员变量
ConfigManager* ConfigManager::instance = nullptr;
std::mutex ConfigManager::mtx;

int main() {
    ConfigManager* configManager = ConfigManager::getInstance();
    std::cout << "Database: " << configManager->getValue("database") << std::endl;
    std::cout << "User: " << configManager->getValue("user") << std::endl;
    return 0;
}
```
### 优缺点
#### 优点：

- 控制实例的数量，确保只有一个实例存在。
- 提供全局访问点，方便访问实例。
- 延迟实例化（懒汉式），节省内存资源。
#### 缺点：
- 实现复杂度增加，特别是需要处理线程安全问题时。
- 不适合需要频繁创建和销毁对象的场景。
- 可能导致单例对象过多，占用内存。
# 工厂模式（Factory Pattern）

### 概述
工厂模式是一种创建型设计模式，定义了一个创建对象的接口，但由子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。

### 什么时候使用工厂模式？
- 需要创建对象，但不知道具体实现类时。
- 创建对象的逻辑复杂，且需要复用时。
- 需要解耦对象的创建和使用时。

### 工厂模式的实现
工厂模式可以分为简单工厂模式、工厂方法模式和抽象工厂模式。

#### 简单工厂模式
简单工厂模式是工厂模式的最简单形式，通过一个工厂类来创建对象。
```cpp
#include <iostream>
#include <string>

class Product {
public:
    virtual void use() = 0;
};

class ConcreteProductA : public Product {
public:
    void use() override {
        std::cout << "Using Product A" << std::endl;
    }
};

class ConcreteProductB : public Product {
public:
    void use() override {
        std::cout << "Using Product B" << std::endl;
    }
};

class SimpleFactory {
public:
    static Product* createProduct(const std::string& type) {
        if (type == "A") {
            return new ConcreteProductA();
        } else if (type == "B") {
            return new ConcreteProductB();
        } else {
            return nullptr;
        }
    }
};

int main() {
    Product* productA = SimpleFactory::createProduct("A");
    productA->use();

    Product* productB = SimpleFactory::createProduct("B");
    productB->use();

    delete productA;
    delete productB;

    return 0;
}
```
#### 工厂方法模式
工厂方法模式定义了一个创建对象的接口，但由子类决定实例化哪一个类。
```cpp
#include <iostream>
#include <string>

class Product {
public:
    virtual void use() = 0;
};

class ConcreteProductA : public Product {
public:
    void use() override {
        std::cout << "Using Product A" << std::endl;
    }
};

class ConcreteProductB : public Product {
public:
    void use() override {
        std::cout << "Using Product B" << std::endl;
    }
};

class Factory {
public:
    virtual Product* createProduct() = 0;
};

class ConcreteFactoryA : public Factory {
public:
    Product* createProduct() override {
        return new ConcreteProductA();
    }
};

class ConcreteFactoryB : public Factory {
public:
    Product* createProduct() override {
        return new ConcreteProductB();
   
```
```cpp
#include <iostream>
#include <string>

class Product {
public:
    virtual void use() = 0;
};

class ConcreteProductA : public Product {
public:
    void use() override {
        std::cout << "Using Product A" << std::endl;
    }
};

class ConcreteProductB : public Product {
public:
    void use() override {
        std::cout << "Using Product B" << std::endl;
    }
};

class Factory {
public:
    virtual Product* createProduct() = 0;
};

class ConcreteFactoryA : public Factory {
public:
    Product* createProduct() override {
        return new ConcreteProductA();
    }
};

class ConcreteFactoryB : public Factory {
public:
    Product* createProduct() override {
        return new ConcreteProductB();
    }
};

int main() {
    Factory* factoryA = new ConcreteFactoryA();
    Product* productA = factoryA->createProduct();
    productA->use();

    Factory* factoryB = new ConcreteFactoryB();
    Product* productB = factoryB->createProduct();
    productB->use();

    delete productA;
    delete productB;
    delete factoryA;
    delete factoryB;

    return 0;
}
```

#### 抽象工厂模式
抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。
```cpp
#include <iostream>
#include <string>

// 产品A的抽象类
class AbstractProductA {
public:
    virtual void use() = 0;
};

// 产品B的抽象类
class AbstractProductB {
public:
    virtual void eat() = 0;
};

// 产品A的具体类
class ProductA1 : public AbstractProductA {
public:
    void use() override {
        std::cout << "Using Product A1" << std::endl;
    }
};

class ProductA2 : public AbstractProductA {
public:
    void use() override {
        std::cout << "Using Product A2" << std::endl;
    }
};

// 产品B的具体类
class ProductB1 : public AbstractProductB {
public:
    void eat() override {
        std::cout << "Eating Product B1" << std::endl;
    }
};

class ProductB2 : public AbstractProductB {
public:
    void eat() override {
        std::cout << "Eating Product B2" << std::endl;
    }
};

// 抽象工厂类
class AbstractFactory {
public:
    virtual AbstractProductA* createProductA() = 0;
    virtual AbstractProductB* createProductB() = 0;
};

// 具体工厂类1
class ConcreteFactory1 : public AbstractFactory {
public:
    AbstractProductA* createProductA() override {
        return new ProductA1();
    }

    AbstractProductB* createProductB() override {
        return new ProductB1();
    }
};

// 具体工厂类2
class ConcreteFactory2 : public AbstractFactory {
public:
    AbstractProductA* createProductA() override {
        return new ProductA2();
    }

    AbstractProductB* createProductB() override {
        return new ProductB2();
    }
};

int main() {
    AbstractFactory* factory1 = new ConcreteFactory1();
    AbstractProductA* productA1 = factory1->createProductA();
    AbstractProductB* productB1 = factory1->createProductB();
    productA1->use();
    productB1->eat();

    AbstractFactory* factory2 = new ConcreteFactory2();
    AbstractProductA* productA2 = factory2->createProductA();
    AbstractProductB* productB2 = factory2->createProductB();
    productA2->use();
    productB2->eat();

    delete productA1;
    delete productB1;
    delete productA2;
    delete productB2;
    delete factory1;
    delete factory2;

    return 0;
}
```

### 实际开发中的工厂模式

#### 日志记录器工厂
在实际开发中，工厂模式常用于创建日志记录器。不同的日志记录器（如文件日志记录器、控制台日志记录器）可以通过工厂方法来创建。
```cpp
#include <iostream>
#include <string>

// 日志记录器接口
class Logger {
public:
    virtual void log(const std::string& message) = 0;
};

// 文件日志记录器
class FileLogger : public Logger {
public:
    void log(const std::string& message) override {
        std::cout << "FileLogger: " << message << std::endl;
    }
};

// 控制台日志记录器
class ConsoleLogger : public Logger {
public:
    void log(const std::string& message) override {
        std::cout << "ConsoleLogger: " << message << std::endl;
    }
};

// 日志记录器工厂
class LoggerFactory {
public:
    virtual Logger* createLogger() = 0;
};

// 文件日志记录器工厂
class FileLoggerFactory : public LoggerFactory {
public:
    Logger* createLogger() override {
        return new FileLogger();
    }
};

// 控制台日志记录器工厂
class ConsoleLoggerFactory : public LoggerFactory {
public:
    Logger* createLogger() override {
        return new ConsoleLogger();
    }
};

int main() {
    LoggerFactory* fileLoggerFactory = new FileLoggerFactory();
    Logger* fileLogger = fileLoggerFactory->createLogger();
    fileLogger->log("File log message");

    LoggerFactory* consoleLoggerFactory = new ConsoleLoggerFactory();
    Logger* consoleLogger = consoleLoggerFactory->createLogger();
    consoleLogger->log("Console log message");

    delete fileLogger;
    delete consoleLogger;
    delete fileLoggerFactory;
    delete consoleLoggerFactory;

    return 0;
}
```
### 优缺点

#### 优点：
- 解耦对象的创建和使用。
- 提高代码的可维护性和可扩展性。
- 可以通过子类决定创建哪一个具体类的实例。

#### 缺点：
- 增加了系统的复杂度，尤其是抽象工厂模式。
- 每增加一个产品类，都需要增加相应的工厂类。

# 观察者模式（Observer Pattern）

### 概述
观察者模式定义了一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并自动更新。

### 什么时候使用观察者模式？
- 当一个对象的状态改变需要通知其他对象时。
- 当一个对象的改变需要触发其他对象的行为，而又不希望这些对象之间存在紧密耦合时

### 观察者模式的实现
观察者模式涉及两个主要角色：主题（Subject）和观察者（Observer）。
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

// 抽象观察者
class Observer {
public:
    virtual void update(const std::string& message) = 0;
};

// 具体观察者
class ConcreteObserver : public Observer {
private:
    std::string name;

public:
    ConcreteObserver(const std::string& name) : name(name) {}

    void update(const std::string& message) override {
        std::cout << name << " received message: " << message << std::endl;
    }
};

// 抽象主题
class Subject {
private:
    std::vector<Observer*> observers;

public:
    void attach(Observer* observer) {
        observers.push_back(observer);
    }

    void detach(Observer* observer) {
        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }

    void notify(const std::string& message) {
        for (Observer* observer : observers) {
            observer->update(message);
        }
    }
};

// 具体主题
class ConcreteSubject : public Subject {
private:
    std::string state;

public:
    void setState(const std::string& newState) {
        state = newState;
        notify(state);
    }

    std::string getState() {
        return state;
    }
};

int main() {
    ConcreteSubject subject;
    ConcreteObserver observer1("Observer 1");
    ConcreteObserver observer2("Observer 2");

    subject.attach(&observer1);
    subject.attach(&observer2);

    subject.setState("State 1");
    subject.setState("State 2");

    subject.detach(&observer1);
    subject.setState("State 3");

    return 0;
}
```

### 实际开发中的观察者模式

#### GUI事件系统
在实际开发中，观察者模式常用于GUI事件系统，当用户在界面上进行操作时，事件处理器（观察者）会接收到通知并做出相应处理。
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>

// 事件处理器接口
class EventHandler {
public:
    virtual void handleEvent(const std::string& event) = 0;
};

// 具体事件处理器
class ButtonClickHandler : public EventHandler {
public:
    void handleEvent(const std::string& event) override {
        std::cout << "Button clicked: " << event << std::endl;
    }
};

// GUI组件
class Button {
private:
    std::vector<EventHandler*> handlers;

public:
    void addEventHandler(EventHandler* handler) {
        handlers.push_back(handler);
    }

    void removeEventHandler(EventHandler* handler) {
        handlers.erase(std::remove(handlers.begin(), handlers.end(), handler), handlers.end());
    }

    void click(const std::string& event) {
        for (EventHandler* handler : handlers) {
            handler->handleEvent(event);
        }
    }
};

int main() {
    Button button;
    ButtonClickHandler clickHandler;

    button.addEventHandler(&clickHandler);
    button.click("Click Event 1");
    button.click("Click Event 2");

    return 0;
}
```

### 优缺点：

#### 优点：
- 解耦了观察者和主题，使得彼此独立变化。
- 提高代码的可维护性和可扩展性。
- 支持广播通信。

#### 缺点
- 如果观察者对象较多，通知所有观察者可能会导致性能问题。
- 观察者之间存在依赖关系时，可能会引发连锁更新，导致系统复杂性增加。

# 总结
单例模式、工厂模式和观察者模式是三种常见的设计模式，每种模式都有其特定的使用场景和优缺点。在实际开发中，合理选择和使用设计模式可以提高代码的可维护性、可扩展性和重用性。通过实际的代码示例，我们可以更好地理解这些设计模式的应用和实现。
