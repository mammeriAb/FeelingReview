# C++ / Qt Exercises

Try each exercise first. Solutions are in [cpp-qt-solutions.md](cpp-qt-solutions.md).

---

## 1. C++ core

### Exercise 1 — Pointer vs reference

Which lines fail to compile? Which are legal? Say why.

```cpp
int x = 1;
int* p = nullptr;
int& r;          // (a)
p = &x;          // (b)
r = x;           // (c)  (assume r was never initialized)
int& r2 = x;     // (d)
r2 = 7;          // (e)
p = nullptr;     // (f)
r2 = nullptr;    // (g)
```

### Exercise 2 — `const` pointers

Fill in the blanks with `yes` or `no`.

```cpp
int n = 0;
const int* a = &n;       // can you do *a = 1?   ___   a = &n? ___
int* const b = &n;       // can you do *b = 1?   ___   b = &n? ___
const int* const c = &n; // can you do *c = 1?   ___   c = &n? ___
```

### Exercise 3 — Find the bug (dangling)

```cpp
const char* name() {
    std::string s = "Qt";
    return s.c_str();
}

int main() {
    const char* p = name();
    std::cout << p;
}
```

What goes wrong? Rewrite `name()` so the caller gets a safe string.

### Exercise 4 — RAII wrapper

Write a tiny `File` class that:

- opens a `FILE*` in the constructor (`fopen`)
- closes it in the destructor
- is not copyable
- is movable

Sketch only; error handling can be a thrown `std::runtime_error` on failed open.

### Exercise 5 — Rule of Five / Zero

This class manages a resource by hand. Rewrite it using Rule of Zero (no custom special members).

```cpp
class Buffer {
    char* data_;
    std::size_t n_;
public:
    Buffer(std::size_t n) : data_(new char[n]{}), n_(n) {}
    ~Buffer() { delete[] data_; }
    Buffer(const Buffer& o) : data_(new char[o.n_]), n_(o.n_) {
        std::copy(o.data_, o.data_ + n_, data_);
    }
    Buffer& operator=(const Buffer& o) { /* ... */ return *this; }
};
```

### Exercise 6 — Predict the output (move)

```cpp
std::string a = "hello";
std::string b = std::move(a);
std::cout << "a=[" << a << "] b=[" << b << "]\n";
```

What is guaranteed about `b`? What is guaranteed about `a`?

### Exercise 7 — Virtual destructor

```cpp
struct Base {
    Base() { std::puts("B"); }
    ~Base() { std::puts("~B"); }
};
struct Derived : Base {
    Derived() { std::puts("D"); }
    ~Derived() { std::puts("~D"); }
};

int main() {
    Base* p = new Derived;
    delete p;
}
```

What is printed? What is undefined? Fix the class.

### Exercise 8 — Override vs hide

```cpp
struct B {
    virtual void f(int) { std::puts("B"); }
};
struct D : B {
    void f(double) { std::puts("D"); }
};

int main() {
    D d;
    B& b = d;
    d.f(1);
    b.f(1);
}
```

What is printed for each call? How do you make `D` actually override `B::f`?

### Exercise 9 — Object slicing

```cpp
struct Animal { virtual const char* name() { return "animal"; } };
struct Cat : Animal { const char* name() override { return "cat"; } };

void print(Animal a) { std::puts(a.name()); }

int main() {
    Cat c;
    print(c);
}
```

What is printed? Change the function signature so it prints `cat`.

### Exercise 10 — `explicit`

```cpp
struct Meter {
    explicit Meter(int v) : v_(v) {}
    int v_;
};
void take(Meter m);

take(10);        // (a)
take(Meter{10}); // (b)
Meter m = 10;    // (c)
```

Which compile? Why is `explicit` useful here?

---

## 2. Modern C++ and STL

### Exercise 11 — Pick a smart pointer

For each situation, choose `unique_ptr`, `shared_ptr`, `weak_ptr`, or a stack object.

1. A widget owned by one window, never shared.  
2. A cache of images used by many views.  
3. A back-pointer from a child node to its parent in a shared-ownership tree.  
4. A `std::mutex` guarding a local critical section.

### Exercise 12 — Cycle leak

```cpp
struct Node {
    std::shared_ptr<Node> parent;
    std::shared_ptr<Node> child;
};

auto p = std::make_shared<Node>();
auto c = std::make_shared<Node>();
p->child = c;
c->parent = p;
```

Why do `p` and `c` leak after both locals die? Fix the design (one-line change of types).

### Exercise 13 — Lambda lifetime

```cpp
std::function<int()> make() {
    int x = 41;
    return [&] { return x + 1; };
}
```

Why is this undefined? Write two correct versions: capture by value, and init-capture.

### Exercise 14 — `vector` invalidation

```cpp
std::vector<int> v = {1, 2, 3};
auto it = v.begin();
v.push_back(4);
std::cout << *it;
```

When is `*it` safe? When is it undefined? How do you make `push_back` safe for existing iterators?

### Exercise 15 — Pass-by

Write the best signature for each:

1. Read a `std::string` without copying (no ownership).  
2. Store a `std::string` inside a class (sink).  
3. A 32-bit id.  
4. A function that may or may not consume a `unique_ptr<Widget>`.

### Exercise 16 — Perfect forwarding

Complete `wrapper` so it forwards `args` to `Widget`’s constructor with the same value category.

```cpp
template<class... Args>
std::unique_ptr<Widget> wrapper(Args&&... args) {
    return std::make_unique<Widget>(/* ??? */);
}
```

### Exercise 17 — Mutex RAII

Rewrite this so the lock is released on every path, including exceptions.

```cpp
g_mutex.lock();
if (error)
    return;
use_shared_data();
g_mutex.unlock();
```

### Exercise 18 — `optional`

Write `find_user(const std::vector<User>& users, int id)` that returns `std::optional<User>` (or `optional<const User*>` if you want to avoid a copy). Show how the caller checks “not found”.

---

## 3. Qt — QObject, signals, slots

### Exercise 19 — Missing `Q_OBJECT`

This class will not work as intended. What is missing, and which symptoms do you get?

```cpp
class Counter : public QObject {
public:
    void setValue(int v);
signals:
    void valueChanged(int);
};
```

### Exercise 20 — Who deletes this button?

```cpp
class Window : public QWidget {
public:
    Window() {
        auto* b = new QPushButton("OK", this);
        auto* c = new QPushButton("No parent");
        auto* d = new QPushButton("Also parented", this);
        delete d; // (1)
    }
};
```

For `b`, `c`, and `d`: who owns them? Is `delete d` OK? What should you do with `c`?

### Exercise 21 — `delete` vs `deleteLater`

Inside a slot `onClicked()`, you want to destroy the window that owns the button. Why is `delete this;` risky? What do you write instead?

### Exercise 22 — Connect it

A `QPushButton* btn` and a `Window` with slot `void onGo()`. Write a compile-time-checked connection. Then write a lambda connection that `qDebug`s `"go"` and is automatically disconnected when `this` dies.

### Exercise 23 — Direct vs queued

Thread A emits `dataReady(int)` on object `w`. Object `w` lives in thread B. Connection type is `Qt::AutoConnection`.

1. In which thread does the slot run?  
2. What if you force `DirectConnection`?  
3. What extra step is needed if the signal argument is `MyStruct`?

### Exercise 24 — Signature mismatch

Which connections are valid?

```cpp
signals:
    void ready(int, QString);
public slots:
    void a(int, QString);
    void b(int);
    void c(int, QString, bool);
    void d(QString, int);
```

### Exercise 25 — Signal to signal

`SearchBox` has `textChanged(QString)`. `App` has `queryChanged(QString)`. Write one connection that forwards the signal without a slot.

### Exercise 26 — Custom type over a queued connection

You have:

```cpp
struct Sample { int hz; QVector<float> data; };
```

List the steps so `emit sampled(s);` can reach a slot in another thread.

### Exercise 27 — `qobject_cast`

```cpp
QObject* o = new QPushButton("Hi");
auto* b = qobject_cast<QPushButton*>(o);
auto* l = qobject_cast<QLabel*>(o);
```

What are `b` and `l`? Why use `qobject_cast` instead of `static_cast` here?

### Exercise 28 — Property

Write a `Q_PROPERTY` for `int count` with getter, setter, and a notify signal. The setter must not emit if the value did not change.

### Exercise 29 — `QString` vs `QByteArray`

You receive UTF-8 bytes from a socket. Convert to `QString`, then back to UTF-8 bytes to send. Write the two calls. Why not `QString::fromLatin1`?

---

## 4. Event loop and widgets

### Exercise 30 — Which application class?

Pick `QCoreApplication`, `QGuiApplication`, or `QApplication`:

1. A CLI downloader using `QNetworkAccessManager` only.  
2. A Qt Quick-only mobile UI.  
3. A `QMainWindow` desktop app.

### Exercise 31 — Event filter

Install an event filter on a `QLineEdit` so that pressing Escape clears the text. Sketch `eventFilter`. Do not subclass `QLineEdit`.

### Exercise 32 — Layout

This UI is wrong. Why? Fix it.

```cpp
auto* w = new QWidget;
auto* layout = new QVBoxLayout;      // not set on w
layout->addWidget(new QPushButton("A"));
w->setGeometry(0, 0, 400, 300);
```

### Exercise 33 — Close event

Override `closeEvent` so that if `m_dirty` is true, you ask the user and can cancel the close. Use `QCloseEvent::ignore()` correctly.

### Exercise 34 — Timer debounce

`QLineEdit::textChanged` fires on every key. Debounce 300 ms, then call `runSearch(text)`. Sketch using `QTimer`.

### Exercise 35 — `update` vs `repaint`

In a custom widget’s `mouseMoveEvent`, you want to redraw a hover highlight. Which call do you use, and where does `QPainter` go?

### Exercise 36 — Nested event loop

```cpp
void MainWindow::onExport() {
    auto* dlg = new QProgressDialog(this);
    dlg->exec();          // user can click File→Export again
}
```

Name two problems. How do you make a safer modal progress dialog?

---

## 5. Model/View

### Exercise 37 — Implement a tiny model

Write a `QAbstractListModel` named `NameModel` that holds `QStringList m_names`. Implement `rowCount` and `data` for `DisplayRole`.

### Exercise 38 — Insert row correctly

You want to append `"Ada"` to that model. Write the `beginInsertRows` / `endInsertRows` sequence. What happens if you skip `beginInsertRows`?

### Exercise 39 — Roles

A view shows an icon and a tooltip per row. Which roles does `data()` need to handle besides `DisplayRole`?

---

## 6. Threading

### Exercise 40 — The classic trap

```cpp
class Worker : public QObject {
    Q_OBJECT
public slots:
    void work() { /* long loop */ }
};

// GUI thread:
auto* t = new QThread;
auto* w = new Worker;
w->moveToThread(t);
t->start();
w->work();   // ???
```

Why does `work()` still run on the GUI thread? Write the correct start (signal or `invokeMethod`).

### Exercise 41 — Constructor affinity

```cpp
class Worker : public QObject {
public:
    Worker() {
        m_timer = new QTimer(this);
        connect(m_timer, &QTimer::timeout, this, &Worker::tick);
        m_timer->start(1000);
    }
};

auto* w = new Worker;
w->moveToThread(thread);
```

Why does the timer not fire on `thread` (or why do you get affinity warnings)? Where should the timer be created?

### Exercise 42 — Stopping

A worker loop runs until cancelled. Sketch a cooperative stop: atomic flag **or** a `stop()` slot, then `quit()` + `wait()` on the `QThread`. Do not use `terminate()`.

### Exercise 43 — GUI from a worker

```cpp
void Worker::run() {
    QMessageBox::information(nullptr, "Done", "Finished");
}
```

Why is this illegal? How do you notify the user?

### Exercise 44 — `processEvents`

```cpp
for (int i = 0; i < 100; ++i) {
    heavyStep(i);
    QCoreApplication::processEvents();
}
```

What can go wrong (reentrancy)? Write a better pattern using a worker + progress signal.

---

## 7. QML

### Exercise 45 — Expose C++ to QML

You have a `QObject` `Counter` with property `count` and slot `increment()`. Give two ways to use it from QML: as a context property named `counter`, and as an instantiable type `App.Counter`.

### Exercise 46 — Forgotten NOTIFY

```qml
Text { text: counter.count }
```

The C++ setter changes `m_count` but does not emit. What does the user see? Fix the setter.

### Exercise 47 — Ownership double-free

```cpp
QObject* Factory::create() {
    return new Item;   // called from QML
}
```

Who owns the returned object by default? How do you keep C++ ownership?

### Exercise 48 — Binding loop

```qml
SpinBox { id: a; value: b.value }
SpinBox { id: b; value: a.value }
```

What happens? Rewrite so A is the source of truth and B only displays/edits it (one model property).

---

## 8. Mixed scenarios (write the design)

### Exercise 49 — Non-blocking file load

A **Open** button should load a large file and fill a `QTableView` without freezing. List the objects, threads, signals, and ownership. Mention how you handle “user closed the window mid-load”.

### Exercise 50 — Find all the bugs

```cpp
class Panel : public QWidget {
    std::unique_ptr<QPushButton> btn_;
    QThread thread_;
    Worker worker_;
public:
    Panel() {
        btn_ = std::make_unique<QPushButton>("Go", this);
        connect(btn_.get(), SIGNAL(clicked()), this, SLOT(onGo(int)));
        worker_.moveToThread(&thread_);
        thread_.start();
    }
    ~Panel() { thread_.terminate(); }
    void onGo() {
        worker_.compute();
        label_->setText(worker_.result());
    }
};
```

List at least **six** independent bugs. You do not need to rewrite the whole class.

### Exercise 51 — Fill in the class

Write a `class Stopwatch : public QObject` with:

- property `int ms` (NOTIFY `msChanged`)
- slots `start()`, `stop()`, `reset()`
- a `QTimer` at 10 ms
- parent-child ownership only (no manual `delete`)

Header sketch is enough.

### Exercise 52 — Translate this UI string

Rewrite so translators see a full sentence, not pieces:

```cpp
label->setText(tr("Hello") + " " + name + tr("!"));
```
