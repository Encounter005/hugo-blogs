+++
title = "建造者模式"
tags = [ "设计模式"]
categories = [ "技术"]
date = "2024-07-13T00:57:29+08:00"
+++
## 简介

建造者模式（Builder Pattern）是一种在软件设计中用来组织对象的构造过程的方法。它将对象的构建和表示分离，允许独立改变构建过程或生成不同的表示形式。这种模式特别适用于那些需要创建复杂对象的场景，在这些场景中，对象的组成部分可能相互依赖或者取决于外部环境条件。

## 建造者模式的主要组件

建造者模式通常包含以下主要组件：

1. **产品（Product）接口**：定义了一个通用的产品或对象结构。所有具体产品的构建都会遵循这个接口。

2. **抽象建造者（Builder Interface）**：为创建特定产品提供一组方法，这些方法用于设置产品的不同部分。该接口是所有具体建造者的超类。

3. **具体建造者（Concrete Builder）**：实现抽象建造者接口，并定义了如何构建具体的产品实例。每个具体的建造者将根据需要构建不同的部分或选择不同的配置来创建产品。

4. **产品工厂（Product Factory）** 或 **导演（Director）**：在一些情况下，可以使用一个中心类来调用特定的建造者方法并构建出一个完整的产品对象。这个类通常不直接与产品的具体实现交互，而是通过调用建造者的接口来完成构建过程。

## 应用场景

建造者模式适用于以下几种情况：

- **复杂对象构造**：当创建对象时涉及多个步骤或依赖于外部环境条件（例如操作系统版本、网络配置等）。
- **可选功能构建**：在产品中包含可选择的组件或选项，如定制电脑配置（CPU类型、内存大小、硬盘驱动器类型等）。

## 实例说明

为了更好地理解建造者模式，我们可以使用一个简单的例子来说明如何应用这个设计模式。假设我们正在组装一台电脑开发一个构建过程，允许用户根据需求自定义电脑的配置

```cpp
#include <iostream>
#include <string>
#include <memory>

// Product
class Computer {
public:
    explicit Computer() = default;
    void setCPU( const std::string &cpu ) { cpu_ = cpu; }
    void setMemory( const std::string &memory ) { memory_ = memory; }
    void setStorage( const std::string &storage ) { storage_ = storage; }
    void print() const {
        std::cout << "CPU:     " << cpu_ << std::endl;
        std::cout << "Memory:  " << memory_ << std::endl;
        std::cout << "Storage: " << storage_ << std::endl;
    }

private:
    std::string cpu_;
    std::string memory_;
    std::string storage_;
};

// Builder interface
class ComputerBuilder {
public:
    virtual ~ComputerBuilder()                       = default;
    virtual void buildCPU( const std::string & )     = 0;
    virtual void buildMemory( const std::string & )  = 0;
    virtual void buildStorage( const std::string & ) = 0;
    virtual std::shared_ptr<Computer> getResult()    = 0;
};

// Concrete Builder
class DesktopComputerBuilder : public ComputerBuilder {
public:
    DesktopComputerBuilder() : computer_( std::make_shared<Computer>() ) {}

    void buildCPU( const std::string &cpu ) override {
        computer_->setCPU( cpu );
    }
    void buildMemory( const std::string &memory ) override {
        computer_->setMemory( memory );
    }
    void buildStorage( const std::string &storage ) override {
        computer_->setStorage( storage );
    }

    std::shared_ptr<Computer> getResult() override { return computer_; }

private:
    std::shared_ptr<Computer> computer_;
};

// Director

class ComputerAssembler {
public:
    std::shared_ptr<Computer> assembleComputer( ComputerBuilder &builder ) {
        builder.buildCPU( "Inter i7" );
        builder.buildStorage( "980 PRO 1TB SSD" );
        builder.buildMemory( "16GB" );
        return builder.getResult();
    }
};

void test_func() {
    DesktopComputerBuilder builder;
    ComputerAssembler assembler;
    auto computer = assembler.assembleComputer( builder );
    computer->print();
}

```

### 分析

在给出的代码示例中，实现了建造者模式的主要组成部分：

1. **产品接口**（`Computer`）：定义了一个用于创建电脑的基本接口。所有的具体产品（如台式电脑、笔记本电脑等）都必须遵循这个接口。

2. **抽象建造者（Builder Interface）**（`ComputerBuilder`）：定义了构建电脑所需的通用方法，比如设置CPU、内存和存储设备等。这些方法对所有具体的建造者类开放，允许它们独立于具体产品来操作构建过程。

3. **具体建造者**（`DesktopComputerBuilder`）：实现了抽象建造者接口并提供了具体实现的方法。每个具体建造者会根据自己的规则和逻辑来执行构建步骤，比如在台式电脑上选择不同的组件配置。

4. **导演类（Director）**（`ComputerAssembler`）：负责调用具体的建造者类进行构建过程。它不直接与产品接口或具体的产品实例交互，而是通过调用抽象建造者的接口方法完成构建。

5. `test_func` 函数展示了如何使用这些组成部分来构建一个台式电脑的例子：

```cpp
DesktopComputerBuilder builder;
ComputerAssembler assembler;
auto computer = assembler.assembleComputer( builder );
computer->print();
```

在这个例子中：

- **Builder** (`DesktopComputerBuilder`) 实例化并执行了具体的构建步骤，比如选择CPU型号、内存大小和存储设备类型。
- **Director** (`ComputerAssembler`) 负责调用 `DesktopComputerBuilder` 的方法来逐步构造电脑。

通过这样的结构设计，建造者模式使得在不同场景下可以有不同的构建过程（不同的具体产品），同时保持了代码的可扩展性和灵活性。例如，我们可以轻松地添加新的构建步骤或创建完全不同的产品类型而无需修改现有的组装逻辑。

## 结论

建造者模式是一种强大的设计模式，尤其在需要创建具有复杂构造过程的对象时非常有用。它通过封装构建过程和分离产品构建逻辑来提高系统的灵活性、可扩展性和可维护性。


## 建造者模式的缺点

尽管建造者模式在很多情况下能够提供强大的对象构造能力，但就像任何设计模式一样，它也有其局限性和潜在的问题：

1. **增加系统复杂性**：引入多个类和接口会增加系统的复杂度。对于那些没有足够需求变化或不需要高度可配置性的项目来说，构建者模式可能会导致过度工程化。

2. **创建过多的实例**：如果建造过程涉及到很多步骤或者每个步骤都有可能产生不同的实例，那么在某些实现中可能会创建大量的对象，这可能导致内存使用和性能问题。优化构建过程以减少不必要的对象创建是很重要的。

3. **代码维护难度**：由于构造过程的细节被封装在多个类中，这对于系统维护来说可能需要更多的关注点。如果建造过程发生变化（例如引入新的组件或改变现有组件的选择逻辑），那么通常需要修改不止一个地方，这可能导致维护和更新的成本增加。

4. **耦合性问题**：尽管抽象建造者试图通过接口来隐藏构建步骤的细节，但具体建造者的实现可能会与产品类紧密关联。这种依赖关系可能使得重构变得困难，并且如果对建造过程进行微小调整，可能需要修改多个相关部分。

5. **代码可读性和理解难度**：建造者模式中的类和方法通常用于封装复杂的构建逻辑，这在一定程度上增加了代码的阅读和理解难度。对于没有熟悉此设计模式的人来说，在初次接触时可能会觉得难以理解整个系统的构造过程。

6. **资源消耗**：如果构建步骤涉及外部依赖或需要大量计算资源（如网络请求、数据库查询等），那么建造者模式可能会增加整体系统对这些资源的需求，从而影响性能和用户体验。

7. **非线性构建路径**：在某些实现中，建造过程可能不是单一线性的，而是有多个选择点。这可能导致难以预测的结果或更复杂的错误处理逻辑。

