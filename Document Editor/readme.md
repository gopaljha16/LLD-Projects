## 🌟 Project-1: Google Docs Clone – Bad Design vs Good Design (with SOLID)

---

## ❌ Bad Design Code Breakdown

### 🔧 Code Summary

This design uses a single `DocumentEditor` class to:

* Add text or image (by guessing from file extension)
* Render the content by checking types at runtime
* Save to a file

This violates multiple SOLID principles:

* ❌ Single Responsibility: It does too many things
* ❌ Open/Closed: Adding new elements like newline/tab breaks existing logic
* ❌ Liskov: No abstraction for interchangeable document elements
* ❌ Dependency Inversion: Depends on file directly, not abstraction

### 🔍 Code Walkthrough

```cpp
class DocumentEditor {
private:
    vector<string> documentElements;  // 🧱 Mixed content: text & images
    string renderedDocument;
```

No abstraction! Everything is just a `string`. We cannot extend or manage properly.

```cpp
void addText(string text) {
    documentElements.push_back(text);
}

void addImage(string imagePath) {
    documentElements.push_back(imagePath);
}
```

Adds raw strings. Images are guessed based on `.jpg/.png`.

```cpp
string renderDocument() {
    for (auto element : documentElements) {
        if (element.substr(...) == ".jpg") // runtime guessing
            result += "[Image: ...]"
        else
            result += element;
    }
}
```

Mixing logic of image/text inside render method — violates SRP.

```cpp
void saveToFile() {
    ofstream file("document.txt");
    file << renderDocument();
}
```

No abstraction. Direct file handling — violates DIP.

---

## ✅ Good Design Code (SOLID Based)

### 🎯 What Changed?

* 🧱 Separated content types using `DocumentElement` interface
* 🪓 SRP: Each class has one job (e.g., render image, save to file)
* 🔁 OCP: We can now add new content (NewLine, Tab, Video, etc.) without breaking code
* 🔌 DIP: Used interface `Persistence` so we can switch storage (DB/File)

---

## 🧱 Class-by-Class Explanation

### 1️⃣ Abstraction: `DocumentElement`

```cpp
class DocumentElement {
public:
    virtual string render() = 0;
};
```

An abstract interface for all content types.

### 2️⃣ Concrete Types

```cpp
class TextElement : public DocumentElement {
    string text;
    string render() override { return text; }
};

class ImageElement : public DocumentElement {
    string imagePath;
    string render() override { return "[Image: " + imagePath + "]"; }
};

class NewLineElement : public DocumentElement {
    string render() override { return "\n"; }
};

class TabSpaceElement : public DocumentElement {
    string render() override { return "\t"; }
};
```

Each class follows SRP and is easily extendable.

### 3️⃣ Document Class

```cpp
class Document {
    vector<DocumentElement*> documentElements;

    void addElement(DocumentElement* e);
    string render() { for each e: result += e->render(); }
};
```

This class **stores content**, not responsible for saving or formatting logic.

### 4️⃣ Persistence Abstraction

```cpp
class Persistence {
public:
    virtual void save(string data) = 0;
};

class FileStorage : public Persistence {
    void save(string data) override {
        write to file
    }
};

class DBStorage : public Persistence {
    void save(string data) override {
        save to database
    }
};
```

This lets us swap file vs DB saving — fulfills DIP!

### 5️⃣ DocumentEditor (Controller)

```cpp
class DocumentEditor {
    Document* document;
    Persistence* storage;

    void addText(string);
    void addImage(string);
    void addNewLine();
    void addTabSpace();
    string renderDocument();
    void saveDocument();
};
```

Acts as a manager/controller. Handles user input and delegates.

---

## 🔁 Code Execution Flow

```plaintext
+-----------------+     uses      +-------------------+     uses      +---------------------+
| DocumentEditor  |  --------->  |    Document        |  --------->  | DocumentElement(s)  |
| - addText()     |              | - addElement()     |              | - TextElement        |
| - renderDoc()   |              | - render()         |              | - ImageElement       |
| - saveDoc()     |                                                      - NewLine, Tab etc.|
+-----------------+              +-------------------+                +---------------------+
         |
         | uses
         v
+-------------------+
|   Persistence     |
+-------------------+
| - save(data)      |
        ^
       / \
      /   \
+-----------+   +--------------+
| FileStorage|   | DBStorage    |
+-----------+   +--------------+
```

---

## ✅ SOLID Principles Recap

| Principle | Applied In                | Explanation                                      |
| --------- | ------------------------- | ------------------------------------------------ |
| SRP       | Each class has one job    | TextElement, FileStorage do one thing only       |
| OCP       | DocumentElement interface | Add new content without modifying render logic   |
| LSP       | All elements replaceable  | All `DocumentElement` subclasses render() safely |
| ISP       | Editor doesn't implement  | Split classes by purpose — not one fat interface |
| DIP       | File/DB via interface     | `Persistence` is abstract, allows decoupling     |

---

