## S.O.L.I.D Principles
The ultimate goal of the developer is not only to make the code executable and generate output but the handle the code in such a way that it is extensible and reusable without breaking when code is extended.

### 1. Single-Responsibility Principle
It says about the ideal way of limiting the responsibities of a Class. Every Class should handle only one responsibility which can have multiple tasks of those responsibility.
Maintaining the class this way helps in Extending the code when required / easy to debug.

```python
# file_manager_srp.py

from pathlib import Path
from zipfile import ZipFile

class FileManager:
    def __init__(self, filename):
        self.path = Path(filename)

    def read(self, encoding="utf-8"):
        return self.path.read_text(encoding)

    def write(self, data, encoding="utf-8"):
        self.path.write_text(data, encoding)

    def compress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="w") as archive:
            archive.write(self.path)

    def decompress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="r") as archive:
            archive.extractall()
```
The above example is from Real Python
The above example have multiple tasks related to different responsibilities that can be performed on a file. The responsibilities are: 1.basic file operations 2.file compression
There are various limitations of the above class.
So the ideal way is to separate the responsibilies as different classes. So that any modification/extension of one class will not impact on other class.

### Open-Closed Principle:
```
Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.
```
Limiting the class' responsibilites is not alone enough to make the class extensible. The Class family should be extensible without modifying the already written class and make the use of extended classes in runtime but not by changing much code in main program.
```python
class Tasks:
    def __init__(self, text, task, **kwargs):
        self.text = text
        self.task = task

    def write_to_file(self):
        if self.task == 'upper':
            return self.text.upper()
        elif self.task == "lower":
            return self.text.lower()
        ...
```
In the above example the responsibility is so tight that if any new task is to be performed on the text then the class needs to be modified by adding new elif. Instead we can make new the class dynamic
```python
from abc import ABC, abstractmethod

class Tasks(ABC):
    def __init__(self, task, **kwargs):
        self.task = task

    @abstractmethod
    def apply_task(self):
        pass

class Lower(Tasks):
    def __init__(self, text):
        super().__init__("lower")
        self.text = text

    def apply_task(self):
        return self.text.lower()

class Upper(Tasks):
    def __init__(self, text):
        super().__init__("upper")
        self.text = text

    def apply_task(self):
        return self.text.upper()
```

### 3. Liskov Substitution Principle (LSP):
```Subtypes must be substitutable for their base types.
```
If new Responsibility is to be added to the family, then extend the Base Class as above by creating new class but not by extending the existing subclass. If you try to change the exisitng subclass, then it might lead to change the types of Base Class and the extended class.

### 4.Interface Segregation Principle (ISP)
In this principle interface means methods and clients means Base class and subclasses. This principle simply says that if a subclass is not implementing all the methods of Base Class, then don't handle those unimplemented classes in subclass rather separate the methods to different Base Classes.
```python
# printers_isp.py

from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print(self, document):
        pass

    @abstractmethod
    def fax(self, document):
        pass

    @abstractmethod
    def scan(self, document):
        pass

class OldPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in black and white...")

    def fax(self, document):
        raise NotImplementedError("Fax functionality not supported")

    def scan(self, document):
        raise NotImplementedError("Scan functionality not supported")

class ModernPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in color...")

    def fax(self, document):
        print(f"Faxing {document}...")

    def scan(self, document):
        print(f"Scanning {document}...")
```
Instead of abovve way, follow the below way
```python
# printers_isp.py

from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print(self, document):
        pass

class Fax(ABC):
    @abstractmethod
    def fax(self, document):
        pass

class Scanner(ABC):
    @abstractmethod
    def scan(self, document):
        pass

class OldPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in black and white...")

class NewPrinter(Printer, Fax, Scanner):
    def print(self, document):
        print(f"Printing {document} in color...")

    def fax(self, document):
        print(f"Faxing {document}...")

    def scan(self, document):
        print(f"Scanning {document}...")
```
### Dependency Inversion Principle (DIP):
```Abstractions should not depend upon details. Details should depend upon abstractions.```
This principle says that the final implementing class should only depend on the type of abstraction it is calling but not on how the dta is handled/returned in each abstraction.
```python
class FrontEnd:
    def __init__(self, back_end):
        self.back_end = back_end

    def display_data(self):
        data = self.back_end.get_data_from_database()
        print("Display data:", data)

class BackEnd:
    def get_data_from_database(self):
        return "Data from the database"
```
Now the Frontend class directly reads the data from Database Class, But what if you want to add another Data Source/API to the frontend. Then you have to change entire Frontend class.
Instead change the Frontend Class in such way that it will display the data irrespective of data source type it uses. The only constraint is that, the data from all the data sources should be in the format as Frontend is expecting.
```python
from abc import ABC, abstractmethod

class FrontEnd:
    def __init__(self, data_source):
        self.data_source = data_source

    def display_data(self):
        data = self.data_source.get_data()
        print("Display data:", data)

class DataSource(ABC):
    @abstractmethod
    def get_data(self):
        pass

class Database(DataSource):
    def get_data(self):
        return "Data from the database"

class API(DataSource):
    def get_data(self):
        return "Data from the API"
```
In the above code, The Frontend is flexible to call any subclass of DataSource abstract class. So we can extend the DatsSource base class as needed without making any changes to the Frontend as long as the data format it receives is consistent.

