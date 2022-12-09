# QtSamples
Many small samples of qt projects

TODO - https://www.youtube.com/watch?time_continue=347&v=iR8K6ya0ZaY&feature=emb_logo


http://qt.shoutwiki.com/wiki/Closing_a_Qt_Quick_application_from_QML


### QtSamples
Test


### WIX

https://stackoverflow.com/questions/36671672/how-to-enable-wix-projects-in-visual-studio-2017

https://marketplace.visualstudio.com/items?itemName=RobMensching.WixToolsetVisualStudio2017Extension

https://github.com/wixtoolset/wix3/releases/tag/wix3104rtm

# Почему не стоит использовать qobject_cast<MyClass*>(sender())

- Этот метод помечен qt как deprecated
- В многопоточных системах с sender() есть проблема:
sender() - вовзвращает отправителя - Последний испустивший сигнал объект
И если повезёт, то sender() - вернёт нам того объекта отправителя сигнала какого нам надо
А если не повезёт, то sender() - вернёт нам последнего, который будет СОВСЕМ НЕ тот кого мы ожидаем
```
MyClass *senderMy   = qobject_cast<MyClass*>(sender());
bool    isSenderMy = (senderMy != nullptr);
```

# Шаблоны и Qt / Q_OBJECT + template<class T> 

https://dolzhenko.blogspot.com/2008/10/qt-usage-c-templates-with-qobject.html
	
**PROBLEM**: Using the Meta-Object Compiler (moc):
moc doesn't create moc class (qt meta info for class) for follow class declaration:
```
template <class T>
class SomeWidget : public SomeSubWidget {
 Q_OBJECT
public:
 SomeWidget(QWidget *parent = 0): SomeSubWidget(parent){
 }
 // for instance, wrap T class
protected:
 virtual T* component(QWidget *parent = 0) { return new T(parent); }

 T *widget;
};
```


**Solution**: devide class for two classes - with Q_OBJECT declaration and with template:
create sub class with Q_OBJECT
```
class SomeSubWidget : public QWidget
{
 Q_OBJECT
public:
 SomeSubWidget(QWidget *parent = 0):QWidget(parent){}
};
```

and when inherit it and add template:
```
template <class T> class SomeWidget : public SomeSubWidget
{
public:
 SomeWidget(QWidget *parent = 0): SomeSubWidget(parent) { } // for instance, wrap T class
protected:
 virtual T* component(QWidget *parent = 0) { return new T(parent); }
 T *widget;
};
```

https://stackoverflow.com/questions/4397478/qt-templated-q-object-class

**PROBLEM**: It is not possible to mix template and Q_OBJECT 

**SOLUTION 2**: If you have a subset of types you can list the slots and signals like this:
```
class SignalsSlots : public QObject
{
  Q_OBJECT
  public:
    explicit SignalsSlots(QObject *parent = 0) :
    QObject(parent) { }
  public slots:
    virtual void writeAsync(int value) { }
  signals:
    void readAsynkPolledChanged(int value);
};
...
template <class T>
class Abstraction : public SignalsSlots
{...
```
	
**Pimpl** 
https://stackoverflow.com/questions/12294981/interfaces-with-template-methods


# QScroller::grabGesture + QScrollerProperties
- QScroller::grabGesture(ui->tableView, QScroller::LeftMouseButtonGesture);
- Approach with QScrollerProperties using the scroller obj.
- setTableScrollerNonDragableProperties<QTreeView>(ui->treeViewTable);

```
#include <QTreeView>
#include <QScroller>
#include <QScrollerProperties>

template <class H>
void setTableScrollerNonDragableProperties(H *table)
{
    QScroller *scrol = QScroller::scroller(table);
    // (*) What this part of code does:
    // When you are at the top and scroll down realy fast on the table with your finger gesture
    // When bable comes to the end it will Drag (move a bit under the visible bottom and return to the top again)
    // To make the table just move down to the end and then simply Stop without dragging and stop at the bottom
    // we prepare QScrollerProperties scrprop; with following parameters:

    QScrollerProperties scrprop;
    QVariant overshootPolicy = QVariant::fromValue<QScrollerProperties::OvershootPolicy>(QScrollerProperties::OvershootAlwaysOff);
    scrprop.setScrollMetric(QScrollerProperties::VerticalOvershootPolicy, overshootPolicy);
    scrprop.setScrollMetric(QScrollerProperties::HorizontalOvershootPolicy, overshootPolicy);
    scrprop.setScrollMetric(QScrollerProperties::DragStartDistance, 0.001);
    scrprop.setScrollMetric(QScrollerProperties::DragStartDistance, 0.001);

    scrol->setScrollerProperties(scrprop);

    scrol->grabGesture(table, QScroller::LeftMouseButtonGesture);
}

```

# QString s; s.arg().arg()... <=> QStringList

```
QStringList listProxy;
listProxy.append("Z");
listProxy.append("X");
listProxy.append("Y");
QString funcProxy = "a'%1'b'%2'c'%3'"; // a'Z'b'X'c'Y'
for (int i = 0; i < listProxy.size(); ++i)
    funcProxy.replace(QString("%%1").arg(i + 1), listProxy.at(i));
```

# QMessageBox

### QMessageBox BAD:
```
QMessageBox msg;
    msg.setWindowTitle(tr("Invalid variable value"));
    msg.setText(tr("Unable to convert Variant value to int state"));
    msg.exec();
```

https://doc.qt.io/qt-5/qdialog.html#exec

### QMessageBox GOOD:
```
QMessageBox::warning(nullptr, "Warning", "Macros recording wasn't stopped!\n Stop recording the macros and try again.", "Ok");
QMessageBox::critical(this, "Error", error);
QMessageBox::about(this, tr("About program"), aboutMessage);

```

# qstring arg multiple times in a circle

	QString str;

	// Version 1
	str = "some %5987%239 random %66 text";
	for(int i=0; i<3; ++i)
	{
		str = str.arg( QString::number( (i+1)*10 ) );
		// returns "some 1020 random 30 text"
	}
	
	// Version 2
	str = "some %1 random %2 text %3 a %3 b %1 c %2 d finish";
	for(int i=0; i<3; ++i)
	{
		QString argNum = "%"+QString::number(i+1);
		str = str.arg(argNum).arg( QString::number( (i+1)*10 ) );
		// returns "some 10 random 20 text 30 a 30 b 10 c 20 d finish"
	}


# tr("your cool text") => ???? ?????? ?????

PROBLEM: Sometimes we get **???? ???? ???** on the place of our translated text.
Mostly It happends when we translate from English to Any Unicode Language.

setText(QCoreApplication::translate("Text", text.toLocal8Bit()));

### toLocal8Bit() can cause this problem

Use instead of toLocal8Bit() <---> toUtf8()


# QFile .open() correct error handling file.errorString()

```
QString flName;
QFile fl(flName); // exists / permission denied / other process busy
QString error;
if(fl.exists())
{
    if(fl.open(QIODevice::ReadOnly))
    {
        QByteArray block = fl.read(10);
        fl.write(block);
        fl.close();
    }
    else
    {
        error = QString("File: %1 Error: %2").arg(flName).arg(fl.errorString());
    }
}
else
{
    error = "File does not exist";
}

```


# QDir create for DEBUG

```
QDateTime dt = QDateTime::currentDateTime();
QDir dr; dr.mkdir("C:/Users/Public/test/" + dt.toString("dd-MM-yyyy-hh-mm-ss-z"));
```

# QFile create for DEBUG

```
{
	QDateTime dt = QDateTime::currentDateTime();
	QString sdt = dt.toString("dd_MM_yyyy_hh_mm_ss_z");
	sdt = sdt.append(".txt");

	QString file = "C:/test/_/" + sdt;

	writeStrToFile("1", file);
	writeStrToFile("2", file);
	writeStrToFile("3", file);
}

void writeStrToFile(QString scriptTetx, QString fileName)
{
	std::string s = fileName.toStdString();

	QFile scriptFile(fileName);
	if (scriptFile.open(QIODevice::ReadWrite))
	{
		// Save the script
		QTextStream outputStream(&scriptFile);
		QString code = scriptTetx;
		code = code.append("\n");
		outputStream.seek(outputStream.readAll().size());
		outputStream << code;
		scriptFile.close();
	}
}
```

# QJSEngine - setObjectOwnership

```
	QJSEngine _scriptEngine(this);
	// Register objects for access from script
	QJSValue svApi = _scriptEngine.newQObject(this);
	_scriptEngine.globalObject().setProperty("api", svApi);

	// all cpp objects that were passed to QJSEngine or QQmlEngine will be destroy by context of these object.
	// Without this call destuctor of ScriptInterface will be called 2 times - by EmulatorUiApplication and ScriptEngine;
	QQmlEngine::setObjectOwnership(this, QQmlEngine::CppOwnership);
```

# QString - smal guide
	
```
    QString s("1234567890");
    s = s.mid(3, s.length() - 3); // all after first 3
    s = s.left(3); // first 3
    s = s.right(3); // last 3
    
```

### SELECT BIG Data from a Large Table CORREC Solution:

```
    //
    // canFetchMore() + fetchMore() approach
    //
    
    while (mModel->canFetchMore())
        mModel->fetchMore();
    int totalRows = mModel->rowCount();
    
    for (int row = 0; row < totalRows; ++row) {
        for (int col = 0; col < mModel.columnCount(); ++col) {
            out << mModel->index(row, col).data().toString() << mFieldDelimeter;
        }
        out << EOL;
    }

```

### SELECT BIG Data from a Large Table BAD Solution:

```
#include <QSqlQueryModel>
#include <QSqlDatabase>
#include <QSqlQuery>
#include <QSqlRecord>
#include <QSqlError>


	int rows = model->rowCount();
        int cols = model->columnCount();
        bool modelShowsAllRecords = !mModel->canFetchMore();
        
        if (modelShowsAllRecords) {
            for (int row = 0; row < rowCnt; ++row) {
                for (int col = 0; col < colCnt; ++col) {
                    out << mModel->index(row, col).data().toString() << mFieldDelimeter;
                }
                out << EOL;
            }
        }
        else {
            QSqlQuery query(mModel->query());
            bool      table_ok = query.exec();

            if (!table_ok) {
                QString sqlError("Error from SystemEventLogTable: " + query.lastError().text());
                qDebug() << sqlError;
                return ExportError(QString("Export to file error: %1").arg(sqlError));
            }
            else {
                // Only way to get a row count, size() method in QSqlQuery class
                // does not work for SQLite3
                query.last();
                int rowFullCount = query.at() + 1;
                if (rowCnt != rowFullCount)
                    rowCnt = rowFullCount;
            }

            if (rowCnt != 0) {
                query.first(); // return to first row
                do {
                    for (int col = 0; col < colCnt; ++col) {
                        out << query.value(col).toString() << mFieldDelimeter;
                    }
                    out << EOL;
                } while (query.next());
            }
        }

        return Result<void>::ok();
```
	

### OLD find function version - NO ALGO used


```
findTrainNumberNumberInContainer( QVector<StationScheduleInfo>& vec, int trainNumber, int& countStops)
{
	bool isNumberFound = false;
	for (auto a : vec)
	{
		if (countStops > 0 && !isNumberFound)
			return true;

		isNumberFound = (a.trainNum() == trainNumber);
		if (isNumberFound)
			countStops++;
	}

	return false;
}

```

## NEW find function version - USING  ALGORUTHM

```
	countStops = std::count_if(vec.begin(), vec.end(), [&](StationScheduleInfo elem) { return elem.trainNum() == trainNumber; });
	return countStops > 0;
```


### Center Coordinates for dialog
```
void GetCenterCoordsForDlg(QRect dlgLoginSize)
{
	// QListIterator<QScreen*> it(qApp->screens());
	// QScreen* firstScreen = qApp->screens().at(0);
	// QRect rct = (firstScreen)->geometry();
	// int sumWidth = rct.width();
	// while (it.hasNext())
	// {
	// 	QScreen *singleScreen = it.next();
	// 	sumWidth += singleScreen->geometry().width();
	// }

	QRect resultRect;

	QRect mainWindowRect = this->geometry();
	QListIterator<QScreen*> it(qApp->screens());
	QScreen* firstScreen = qApp->screens().at(0);
	QRect rct = (firstScreen)->geometry();
	int sumWidth = rct.width();
	// Show login dialog upon top-left x coordinate
	//int appWindowXposition = this->x();
	// Show login dialog upon coordinates of the center of the application window
	int appWindowXposition = this->x() + this->width() / 2;
	while (it.hasNext())
	{
		QScreen *singleScreen = it.next();
		QRect screenRect = singleScreen->geometry();
		int lastFoundScreenWidth = screenRect.width();

		if (appWindowXposition < sumWidth)
		{
			auto height = qMin(dlgLoginSize.height(), screenRect.height());
			auto width = qMin(dlgLoginSize.width(), screenRect.width());
			auto x = (screenRect.width() - width) / 2;	// center
			x += (sumWidth - lastFoundScreenWidth);		// center + step to right on all of the screens that we have met before
			auto y = (screenRect.height() - height) / 2;

			resultRect = QRect(x, y, width, height);
			qApp->sendCenterCoordsForDlg(resultRect);
		}
		else
		{
			sumWidth += screenRect.width();
		}
	}

}
```

# BEFORE REVIEW

```

QVariant StationScheduleTableModel::data(const QModelIndex & index, int role) const
{
  if (index.isValid())
  {
    int irow = index.row();
    int cnt = _itemsShown.count();

    if (irow < 0 || irow >= cnt)
    	return QVariant();

    const StationScheduleInfo &item = _itemsShown[irow];

    switch (role)
    {
    case TrainNumberRole:
    {
    	if (irow % _distanceNum == 0)
    		return item.trainNum();
    }
    break;
    case StationIdRole:
    	return item.stationId();
    case StationNameRole:
    	return item.stationName();
    case ArivalTimeRole:
    	return item.arivalTimeString();
    case DepartureTimeRole:
    	return item.departureTimeString();
    default:
    	return QVariant();
    }
  }

  return QVariant();
}

```

# AFTER REVIEW

```
  QVariant result;
  if (index.isValid())
  {
    int irow = index.row();
    int cnt = _itemsShown.count();

    if (irow < 0 || irow >= cnt)
    	return QVariant();

    const StationScheduleInfo &item = _itemsShown[irow];

    switch (role)
    {
    case TrainNumberRole:
    {
    	if (irow % _distanceNum == 0)
    		result = item.trainNum();
    }
    break;
    case StationIdRole:
    	result = item.stationId();
    	break;
    case StationNameRole:
    	result = item.stationName();
    	break;
    case ArivalTimeRole:
    	result = item.arivalTimeString();
    	break;
    case DepartureTimeRole:
    	result = item.departureTimeString();
    	break;
    default: break;
    }
  }

  return result;
```

```
{
    connect(ui->slider, &QSlider::valueChanged, this, &A::onSetValue);
    connect(ui->stepLeftButton, &QPushButton::clicked, this, &A::onMoveStepLeft);
    connect(ui->stepRightButton, &QPushButton::clicked, this, &A::onMoveStepRight);
    ui->slider->setValue(settings->value());
    ui->label->setText(QString::number(settings->laserValue()));

    ui->slider->setSingleStep(STEP);
    ui->slider->setPageStep(STEP);
    ui->slider->setTickInterval(STEP);
}
void A::onSetValue(int v)
{
    int res = std::round(v / STEP) * STEP;
    ui->slider->setValue(res);
    settings->setValue(res);
    ui->label->setText(QString::number(res));
}
void A::onMoveStepLeft()
{
    doMoveStep(true);
}
void A::onMoveStepRight()
{
    doMoveStep(false);
}

void A:doMoveStep(bool isleft)
{
    int v   = ui->slider->value();
    auto mi = ui->slider->minimum();
    auto ma = ui->slider->maximum();
    auto s  = STEP;

    for (int i = mi, l, r; i < ma; i += s) {
        l  = i;
        r = l + s;
        if (v >= l && v <= r) {
            if (l == v && v >= m) {
                v = isleft ? (l - s) : (l + s);
                break;
            }
            else if (r == v && v <= ma) {
                v = isLeft ? (r - s) : (r + s);
                break;
            }
            else if (v > l && v < r) {
                val = isleft ? l : r;
                break;
            }
        }
    }
    ui->slider->setValue(v);
}
```


# **Разработка веб-приложений с использованием QML и Qt для WebAssembly**

 **Установка и настройка.**

Нужно сделать две вещи: настроить компилятор Emscripten и Qt для веб-сборки.

**Emscripten**

на Linux:

- `# fetch Emscripten SDK and install target version`
- `git clone https://github.com/emscripten-core/emsdk.git`
- `cd emsdk`
- `./emsdk install sdk-1.38.30-64bit`
- `./emsdk activate --embedded sdk-1.38.30-64bit`
- `source emsdk_env.sh`

Обратите внимание, что 1.38.30 рекомендуется для сборки, не экспериментируйте с другими сборками, так как у них будут проблемы.

Исходный код `emsdk_env.sh` важен для настройки правильной среды *Emscripten*, поэтому не забывайте запускать ее <u>(каждый раз, когда вы открываете новый терминал</u>)


**Qt WebAssembly**

Это еще проще. Загрузите установщик [Qt со страницы загрузки Qt](https://www.qt.io/download) или откройте существующий [инструмент Qt Maintenance,](https://doc.qt.io/qt-5/gettingstarted.html#updating-or-removing-qt) если он у вас уже установлен.

В обоих случаях вам просто нужно выбрать Qt для WebAssembly, в версии Qt  от 5.13.2.

 

![https://miro.medium.com/max/1633/1*XF9KMNHHaFMSjoUS6FbNpA.jpeg](file:///C:/Users/IGOR~1.SHA/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

Установка Qt WebAssembly



# Привет WebAssembly

Перейдите к вашему проекту и создайте `build `каталог.  Затем просто вызовите новый `qmake `с поддержкой WebAssembly:

- `cd /path-to-hello-webassembly-project` - преход в дирректорию проекта
- `mkdir build && cd build` - создание папки `build` и переход в неё
- `/path-to-hello-webassembly/5.13.2/wasm_32/bin/qmake ..` - компилирует проект
- `make`

После того, как это будет сделано, вы получите один из файлов в вашем каталоге сборки -  projectname.html. Чтобы запустить его в браузере, нам нужен, как обычно, веб-сервер. Вы можете использовать все, что захотите, для обслуживания этих статических файлов, но есть также удобный инструмент командной строки emrun, который вы можете использовать для этой цели, поскольку он уже доступен как часть среды Emscripten.

Итак, чтобы открыть наше приложение в браузере Firefox, можно сделать:

- `emrun --browser=firefox *.html`
 

**<u>Пример из dev-shaldunov:</u>**

1. Выполняем из /home/user/emsdk:

     `./emsdk activate --embadded sdk-1.38.30-64bit`

2. `source emsdk_env.sh`
3.  Создаем папку `build` в корне проекта: /mywebassembly/build:
4. `/opt/Qt/5.13.2/wasm_32/bin/qmake ..`
5. `make` 

6. Чтобы открыть наше приложение в браузере Firefox, можно сделать:

   `emrun --browser=firefox *.html`

   

   P.S.: Более подробное описание Вы можете найти на: https://itnext.io/developing-web-apps-using-qml-and-qt-for-webassembly-aa84453f2f61
   


# ООП

### Принципы ООП
1) Инкапсуляция - объект контролирует изменение своих свойств - это механизм языка, позволяющий **объекдинить данные и методы**, работающие с этими данными в единый объект, и **скрыть детали реализации** от пользователя при помощи **модификаторов доступа**. При этом пользователь может выполнять только ограниченный набор операций со приватными членами класса, используя публичные методы.
2) Наследование - позволяет определить класс, имеющий определенный набор характеристик (например, методы и переменные экземпляра), а затем создать другие классы, производные от этого класса. Производный класс наследует поля и функции родительского класса и в зависимости от модификаторов доступа самого наследования, полей и методов имеет к ним доступ.
- **Множественное наследование** является особенностью C++, где один класс может наследовать объекты и методы нескольких классов. Конструкторы наследуемых классов вызываются в том же порядке, в котором они перечислены в строке наследования.
3) Полиморфизм - Различное поведение классов наследников, через единый интерфейс базового класса. 
- Статический - на этапе компиляции - Шаблоны и Перегрузка функций и операторов.
- Динамический - на этапе выполнения - Виртуальные функции. Точное значение операции определяется объектом для которого она вызывается.

### Множественное наследование и построение объекта
```
struct C : ABase, BBase
{
  BField b;
  AFirld a;
}
```
- Конструкторы ABase и BBase вызовутся ранее конструкторов полей "b" и "a" и констр. C
- конструктор вызовется у полей b - он раньше объявлен, a - он позже;
- сначала деструкторы Сам класс, далее сначала поле a, потом поле b и только потом родительсвие опять же в обратном порядке ~BBase ~ABase

### Виртуальное наследование - зачем нужно:
- Решает проблему ромбовидного наследования
```
class A { int x; };
class B: public A {};
class C: public A {};
class D: public B, public C {};  
D d;
d.x; // проблема, из "x" брать из B или из C
     // в D будет 2 экземпляра "х" 1. B::x и 2. C::x
```
РЕШЕНИЕ 1 - Виртуальное наследование
```
class A { int x; };
class B: virtual public A {};
class C: virtual public A {};
class D: public B, public C {};  
D d;
d.x; // проблемы НЕТ из "x" стало одно A::x
```
- Формально пример ниже, не решает проблему ромбовидного наследования, НО Доступ может быть обеспечен через Двойное двоеточие - "оператор разрешения области видимости" к конкретному х, однако факт того, что их всё равно осталось 2 штуки не изменится
```
class A { int x; };
class B: public A {};
class C: public A {};
class D: public B, public C {};  
D d;
d.B::x;
d.C::x;
```
	

https://tproger.ru/articles/move-semantics-and-rvalue/

===================================================================
### Для чего нужен стек ?
- Стек
- Стек представляет собой непрерывную область памяти, адресуемую регистрами ESP (указатель стека) и SS (селектор сегмента стека).
- Особенность стека заключается в том, что данные в него помещаются и извлекаются по принципу «первым вошел – последним вышел». 
- Данные помещаются в стек с помощью инструкции PUSH (заталкивание), а извлекаются по инструкции POP (вытаскивание).
Кроме явного доступа к стеку с помощью инструкций  PUSH и POP, стек автоматически используется процессором при 
выполнении инструкций вызова, возвратов, входа и выхода из процедур, а также при обработке прерываний.

Стек используют для разных целей:
- организации прерываний, вызовов и возвратов;
- временного хранения данных, когда под них нет смысла выделять фиксированные места в памяти;
- передачи и возвращения параметров при вызовах процедур.

**Стек и Куча разница**. Разница между Стеком и Кучей между Кучей и Стеком
1. В стеке распределение и освобождение выполняются центральным процессором и выполняется автоматически, тогда как в куче программист должен делать это вручную.
2. Обработка кадров кучи обходится дороже, чем обработка кадров стека.
3. Реализация стека сложна. В отличие от этого, реализация кучи проста.
4. Вызов функции в стеке занимает O (N) времени. В отличие от этого, требуется куча O (1) времени.
5. Реализация стека в основном страдает от нехватки памяти. Напротив, основной проблемой в куче является фрагментация.
6. Доступ к кадру стека проще, чем кучи, поскольку стек ограничен небольшой областью памяти и всегда попадает в кэш, но кадры кучи разбросаны по всей памяти, поэтому доступ к памяти может привести к большему количеству пропусков кеша.
		
| Сравнение		| стек					| куча
|-----------------------|---------------------------------------|----------------------------------------
| основной		| Память выделяется 			| Память выделяется в случайном порядке
|			| в (LIFO) режиме «посл.вх.перв.вых» 	| - 
| Распределение		| автоматическая 			| Руководство по эксплуатации и выделению
| Стоимость		| Меньше				| Больше
| Реализация		| Жесткий				| Легко
| Вызов фции		| О(N)					| O(1)
| Проблемы		| Ограниченная память			| Фрагментация памяти
| Место-дение ссылки	| Превосходно				| адекватный
| гибкость		| Фиксированный размер и не гибкий	| Изменение размера возможно
| Время доступа		| Быстрее				| Помедленнее
	
===================================================================
### Стек 1 на программу ?

Ответ: Программа это процесс -> у процесса может быть несколько потоков. У каждого потока свой СТЕК (1 юзер-мод стек + 2-ой Кернел мод стэк)

===================================================================

### Процесс от потока - разница ? (один есть основной поток)
- Процесс это контейнер для потоков.
- Несколько потоков пользуются общим адреснымс пространством процесса
	
===================================================================
### static_cast и dynamic_cast разница -> 
- dynamic_cast делает проверку на соотвествие типов
- static_cast НЕ делает проверку на соотвествие типов
	
===================================================================
### фабричный метод = виртуальный конструктор

===================================================================
### make_shared()
https://habr.com/ru/post/509004/

process(std::shared_ptr<Bar>(new Bar), foo());

- Плюс make_shared:
Для: std::shared_ptr<int> x(new int);

Выделений памяти фактически будет два:
- 1-ое в скобках конструктора shared_ptr-а, сам вызов "new int" для выделения памяти под объект
- 2-ое для выделения памяти под сам умный указатель, где оне будет храниить счётчик ссылок int count_copy; и Инкапсулированный указатель на объект Temp*ptr;

Для: std::shared_ptr<int> x(new int);
	
These functions will typically allocate more memory than sizeof(T) to allow for internal bookkeeping structures such as the reference counts

### 1. ПРОБЛЕМА: Может произойти утечка памяти, для случая, 
- СИТУАЦИЯ: когда  функция принимает как минимум два параметра, один из которых инициализируется свежесозданным безымянным std::shared_ptr, а второй параметр инициализируется вызовом другой функции, которая может бросать исключения. 
- СУТЬ: Компилятор не гарантирует порядок "вычисления" аргументов функции, он может перемешать и ЕСЛИ порядок такой - new Bar; foo(); конструктор std::shared_ptr, а не new Bar; к-р std::shared_ptr; foo(), при исключекнии  утечку экземпляра Bar.

А чтобы потенциальная утечка памяти реализовалась — надо ещё два условия:
- чтобы компилятор перемешал вызовы неблагоприятным образом;
- чтобы функция, вычисляющая второй параметр, действительно бросила исключение.

===================================================================

### Хеш таблицы - Разрешение коллизий (Бакеты (+в цепочках Entries ещё)в обих случаях юзаются)
1. **Метод цепочек** - Разрешение коллизий при помощи цепочек.
Каждая ячейка массива H является указателем на связный список (цепочку) пар ключ-значение, соответствующих 
одному и тому же хеш-значению ключа. Коллизии просто приводят к тому, что появляются цепочки длиной более одного элемента.
Операции поиска или удаления элемента требуют просмотра всех элементов соответствующей ему цепочки, 
чтобы найти в ней элемент с заданным ключом. Для добавления элемента нужно добавить элемент в 
конец или начало соответствующего списка, и, в случае, если коэффициент заполнения станет слишком велик, 
увеличить размер массива H и перестроить таблицу.
При предположении, что каждый элемент может попасть в любую позицию таблицы H с равной вероятностью 
и независимо от того, куда попал любой другой элемент, среднее время работы операции поиска элемента 
составляет Θ(1 + α), где α — коэффициент заполнения таблицы.
2. **Открытая адресация**
Пример хеш-таблицы с открытой адресацией и линейным пробированием, получающейся при вставке элементов в левой колонке сверху вниз.
В массиве H хранятся сами пары ключ-значение. Алгоритм вставки элемента проверяет ячейки 
массива H в некотором порядке до тех пор, пока не будет найдена первая свободная ячейка, 
в которую и будет записан новый элемент. Этот порядок вычисляется на лету, что позволяет 
сэкономить на памяти для указателей, требующихся в хеш-таблицах с цепочками.
Последовательность, в которой просматриваются ячейки хеш-таблицы, называется последовательностью проб.

===================================================================

### Лямбды lambda:
**ВАЖНО** - Нужно следить за жизнью лямда выражения и за временем жизни того, что она использует. Ибо она продолжит работать с объектами, которые уже не существуют, то мы столкнёмся с потенциальными проблемами

В C++ 11 лямбда-выражение - это удобный способ определения объекта анонимной функции (замыкания) непосредственно в расположении, где оно вызывается или передается в качестве аргумента функции

```
- Хороши для // Хорошо: Мы можем хранить лямбду в именованной переменной и передавать её в функцию в качестве параметра
auto isEven{   [](int i) { return ((i % 2) == 0); } };
return std::all_of(array.begin(), array.end(), isEven);
```

```
- И примерно такого же аналога в Qt для функции connect -> connect(this, someSig, this, []{ slot-logic right away })
[ captureClause ] ( параметры ) -> возвращаемыйТип
{ стейтменты; }
Поля captureClause и параметры могут быть пустыми, если они не требуются программисту.
Поле возвращаемыйТип является опциональным, и, если его нет, то будет 
использоваться вывод типа с помощью ключевого слова auto.
```
- **"Замыкание"**
	
	auto found{ std::find_if(arr.begin(), arr.end(), [search](std::string_view str) {
	
Поле capture clause используется для того, чтобы предоставить (косвенно) лямбде доступ к переменным из окружающей области видимости, 
к которым она обычно не имеет доступ. Всё, что нам нужно для этого сделать, так это перечислить в поле capture clause объекты, 
к которым мы хотим получить доступ внутри лямбды. В нашем примере мы хотим предоставить лямбде доступ к значению переменной search, поэтому добавляем её в захват

- **"Замыкание" / "Захват"** 
1)  Ключевой момент: Переменные, захваченные лямбдой, являются клонами переменных из внешней области видимости, а не фактическими «внешними» переменными.
2) Для продвинутых читателей: Когда компилятор обнаруживает определение лямбды, он создает для нее определение как для пользовательского объекта. 
3) Каждая захваченная переменная становится элементом данных этого объекта. Во время выполнения программы, при обнаружении определения лямбды, создается экземпляр объекта лямбды и в этот момент инициализируются члены лямбды.
- **Захват по значению**
Чтобы разрешить изменения значения переменных, которые были захвачены по значению, мы можем пометить лямбду как mutable. 
В данном контексте, ключевое слово mutable удаляет спецификатор const со всех переменных, захваченных по значению:
- **Захват по ссылке** - Чтобы захватить переменную по ссылке, мы должны добавить знак амперсанда (&) к имени переменной, которую хотим захватить.
    - В отличие от переменных, которые захватываются по значению, переменные, которые захватываются по ссылке, не являются 
    - константными (если только переменная, которую они захватывают, не является изначально const)

```
int  a = 0;
int  b = 0;
auto f = [a, &b](int m, int &n) {
    // a, b внутри [] являются клонами переменных
    int z = 0;
    z++;

    // a++; // **Захват по значению** ERROR read-only variable
    z = z + a; // ok

    b++; // Ok **Захват по ссылке**

    z = z + m;
    n = z;
};
```
	
===================================================================

- **override** - мы именно перегрузили метод, который был объявлен как virtual
- **finale** - нужен затем, чтобы никто, кто наследуется от этого класса не мог бы перегрузить его

===================================================================

### Размер объекта
Размер объекта в общем случае равен = (1) + (2) + (3) + (4) 

- МЕТОДЫ НЕ ВЛИЯЮТ НА РАЗМЕР ОБЪЕКТА !!! Ответ: 
1) (1) + sizeof всех членов класса 
2) (2) + остаток для выравнивания (по умолчанию выравнивание 4 байта) 
3) (3) + vtblptr = sizeof указателя на vtable (если есть виртуальные функции, 
который находится вскгда ПЕРВЫМ в памяти объетка, чтобы имея указатель на базовый класс 
ты в любом случае мог бы обратиться ко всем функциям (любой из на любой ступни наследования) ) 
4) (4) + указатели на классы предков, от которых было сделано виртуальное наследование 
(=размер указателя * количество классов, есть множ.насл. от двух классов, значит и будет 2 указат. на них) 
(0) - методы внутри объекта не влияют на размер объекта (static) - static-поля НЕ ЯВЛЯЮТСЯ ЧАСТЬЮ КЛАССА
```
### Пустой объект
Занимает минимум 1 байт, чтобы любой объект можно было отличить друг от друга

===================================================================
### lvalue и rvalue
https://tproger.ru/articles/move-semantics-and-rvalue/
- lvalue - то, что имеет конкретный адрес в памяти
- rvalue - то, что адреса не имеет, то, что называется временным объектом
- lvalue в том, что объекты rvalue могут быть перемещены, тогда как объекты lvalue всегда копируются.

===================================================================
### null_ptr
Rengen, для обнуление указателя лучше использовать nullptr, т.к.:
"Причина введения nullptr в язык С++ заключается в устранение возможных недоразумений между (макросом NULL = 0) литералом 0 как целочисленным значением и указателем. Двойное значение литерала 0 может вызывать проблемы при некоторых обстоятельствах. Литерал nullptr имеет тип std::nullptr_t, и его нельзя перепутать со значением другого типа. Он может быть неявно преобразован в любой ссылочный тип, но ни в один целочисленный тип, кроме типа bool." - Айвор Хортон, Visual C++ 2010, Глава 4 *примечание.

Для C++ более старых версий nullptr легко добавляется одной строчкой:
C++Выделить код static struct nullptr_t {template<typename T> operator T* () const {return 0;}} nullptr;
	
### NULL
Страуструп описывал про макрос NULL:
- 5.1.1. Ноль.
В языке С было очень популярно определять макрос NULL для представления такого нулевого указателя. Так как в C++ типы проверяются более жестко, использование банального нуля вместо NULL приведет к меньшим проблемам. Если вы чувствуете, что просто обязаны определить NULL, воспользуйтесь
const int NULL = 0;

===================================================================
```
https://ravesli.com/urok-195-std-weak_ptr-tsiklicheskaya-zavisimost-s-std-shared_ptr/
https://eax.me/cpp-smart-pointers/
https://habr.com/ru/post/140222/
```
- **auto_ptr** - deprectaed, заменён unique_ptr-ром - реализует разрушающее копирование, тоесть имея 2 указателя и при передаче ссылки на объект из одного указателя в другой. Второй куда произошла передача станет "владельцем объекта", а Исходный указатель становится невалидным.
- **unique_ptr** - когда ресурс памяти не должен был разделяемым (у него нет конструктора копирования), но он может быть передан другому unique_ptr или владение им с помощью std::move. представляет собой уникальный указатель на объект. Указатель нельзя копировать, но можно передавать владение им с помощью std::move. При уничтожении указателя автоматически вызывается деструктор объекта, на который он указывает.
- **shared_ptr** - должен использоваться, когда ресурс памяти должен быть разделяемым. (переменная, которая хранит количество указателей, которые ссылаются на объект, Если эта переменная становится равной нулю, то объект уничтожается. Счетчик инкрементируется при каждом вызове либо оператора копирования либо оператора присваивания)
- **weak_ptr** - содержит ccылку на объект, которым управляет shared_ptr, но не производит подсчет числа ссылок; позволяет избавиться от циклической зависимости
- (!) Недостатком умного указателя std::weak_ptr является то, что его нельзя использовать напрямую (нет оператора ->). Чтобы использовать std::weak_ptr, вы сначала должны конвертировать его в std::shared_ptr (с помощью метода lock()), а затем уже использовать std::shared_ptr.
===================================================================

### std::move и std::forward - https://habr.com/ru/post/568306/

```
template<class T> class vector {
    // ...
	Конструктор
	Деструктор
    vector(const vector&);            // copy constructor - конструктор копии
    vector(vector&&) noexcept;        // move constructor - перемещающий конструктор копии
    vector& operator=(const vector&); // copy assignment - оператор присваивания
    vector& operator=(vector&&);      // move assignment - перемещающий оператор присваивания
  public:
  A();
  ~A();
  A(const& A _obj);
  const& A operator=(const& A _obj);
  A(const&& A _obj);
  const& A operator=(const&& A _obj); 	
};
```
- **std::move()** - не выполняет никаких перемещений - занимется ТОЛЬКО ПРИВЕДЕНИЕМ ЛЮБОГО ТИПА к типу rvalue-ссылки. Внутри него происходит
1. Удаление Всех ссылок из входного типа, и
2. Подставление "Двух ампесантов" соотвественно, чтобы получилась Rvalue-ссылка.
- **std::forward()** - фция нужная нам из-за особенности std::move - так как std::move безусловно приводит свой результат к rvalue, а мы хотели бы иметь возможность работкть так же и с lvalue, чтобы они передавались как lvalue
1. Если ссылка была передана как rvalue, то отрабатывает фактически std::move, и передача rvalue – происходит и далее как rvalue
2. И соотвественно если на вход получили lvalue ссылку, то и возвращаем так же lvalue ссылку (просто возвращаем то, что передали нам на вход).
```
template<typename T>
T&& forward(T&& param)
{
    if (is_lvalue_reference<T>::value)
        return param;
    else
        return move(param);
}
```

### **Склейка ссылок**

``` Если при разворачивании шалона, получается lval-ref на lval-ref => 1. T& & => T& 2. T& && => T& 3. T&& & => T& 4. T&& && => T&& ```

===================================================================
### Для чего нужен std::forward?
Функция std::forward, как известно, применяется при идеальной передаче (perfect forwarding).

Идеальная передача позволяет создавать функции-обертки, передающие параметры без каких-либо изменений (lvalue передаются как lvalue, а rvalue – как rvalue) 
и тут **std::move** нам не подходит, так как она безусловно приводит свой результат к rvalue.
Поэтому, была разработана функция **std::forward**, которая выполняет примерно следующую работу:
То есть, если ссылка была передана как rvalue, то вызываем духов std::move, а иначе просто возвращаем то, что передали.

1. Если необходимо «передать» умный указатель **std::unique_ptr**, то сделать мы можем это только через std::move
2. использование std:move можно найти в различных алгоритмах, где нужно менять элементы местами
3. Быстрый Переброс данных из вектора в вектор
```
v2.insert(v2.end(), std::make_move_iterator(v1.begin() + 7), 
                    std::make_move_iterator(v1.end()));
As others have pointed out, first vector v1 is now in indeterminate state, so use erase to clear the mess:
v1.erase(v1.begin() + 7, v1.end());
```
===================================================================
### explicit - https://habr.com/ru/post/436296/
- это ключевое слово ставится перед объявлением конструктора с одним параметром (или с большим числом параметров, 
но когда все параметры, начиная со второго, имеют значения по умолчанию) и предотвращает неявное преобразование типов при инициализации.
- С появлением с C++11 универсальной инициализации explicit обрёл смысл для конструкторов с 0,2,3 и более параметров:
- Помимо этого, начиная с C++11 ключевое слово explicit может также применяться к операторам преобразования типа, также запрещая их неявный вызов:
- В заключение хочется порекомендовать использовать универсальную инициализацию в любом новом коде на C++, а также явно объявлять конструкторы explicit всегда, кроме случаев, когда неявное преобразование семантически оправдано.
===================================================================
### placement new
- Как можно создать объект класса по определённому адресу в памяти ? по конкретному адресу в памяти ?
- Использовать оператор placement new
- Использовать MEMCPY ( Адрес_Куда, Адрес_Откуда, Размер_Скок_Скопировать ) 
``` memcpy( &dst[dstIdx], &src[srcIdx], numElementsToCopy * sizeof( Element ) ); ```
ещё как-то побитово копировать
===================================================================
### mutable 
- Для CONST-тантных методов, которые не могут изменять состояние экземпляра класса и которые и было введено ключевое слово mutable - ибо их он изменять может в отличие от всех сотальных.
- Самый релевантный пример его использования в реальной практике это mutable mutex внутри констатных Get-методов класса, которые должны обратиться к некому общему ресурсу внутри класса.
===================================================================
### static 
- static-методы класса имеют доступ только к статическим полям класса, т.к.
- static-методы не имеют прямой связи с конкретным экземпляром класса
и по этой причине static метод в классе не имеет доступа к указателю this
- inline функции не могут быть помечены как static
===================================================================
### inline
- Это такая функция, чье тело подставляется в каждую точку вызова, вместо того, чтобы генерировать код вызова. Нам не всегда гарантируется её встраиваемость:
- !!! ЕСТЬ ОГРАНИЧЕНИЯ:
- Некоторые компиляторы не могут компилировать встроенный код для функции, содержащей циклы, а также предложения switch и goto.
- Зачастую бывает нельзя создать рекурсивную inline функцию.
- Запрещается использование статических членов функции.
===================================================================
### ключевое слово const ?
1) Позволяет задать константность объекта (Чтобы не было возможности изменить ОБЪЕКТ)
2) Позволяет задать константность указателя (Чтобы не было возможности изменить указатель)
3) Константы и аргументы/результаты функций, а так же Позволяет указать, что данный метод не модифицирует члены класса, т.е. сохраняет состояние объекта. Чтобы метод не мог изменять члены класса, ЗА Исключением членов mutable.

- const int i(1);
i = 2; // ERROR;

- const <-- VALUE IS CONST (cip=&v Ok) <-- |*| --> (*ipc=v Ok) PTR IS CONST --> const

```
{
  int        a   = 1;
  int        b   = 2;
  int        aa  = 11;

  // [CONST INT <=> INT CONST] POINTER
  const int *cip = &a;
  int const *icp = &b;
  //*cip         = aa;  // compile ERROR // assignment of read-only location ‘cip’
  cip = &k;             // Ok

  int        c   = 3;
  int        cc  = 33;
  
  // [CONST INT <=> INT CONST] POINTER
  int *const ipc = &c;
  *ipc           = cc;  // Ok
  //ipc          = &cc; // compile ERROR // assignment of read-only variable ‘ipc’
  
  // ALSO 1
  const char * const s = "data"; // работает
  // ALSO 2
  int i = 4; 
  int* const j(&i); 
}
```

- CONST-метод класса
Помним про const-getter-ы которым может быть нужен mutable mutex для многопоточных операций
```
class A {
private:
  int x;
public:
  void f(int a) const {
    x = a; // <-- не работает
  }
};
```

### Хотим сложную структуру сделать ключём map/set или unordnered_map/set
		    
- Для MAP/SET

https://dawnarc.com/2019/09/c-how-to-use-a-struct-as-key-in-a-std-map/
		    
Переопределение operator < / operator< / operator меньше (и operator==)
```
// Works // Old but Gold
bool compareFunc(const Param& lhs, const Param& rhs)
{
  bool res = false;
  if (lhs.param_1 < rhs.param_1)
  { res = true; }
  else if(lhs.param_1 == rhs.param_1)
  {
    if (lhs.param_2 < rhs.param_2 )
    { res = true; }
    else if(lhs.param_2 == rhs.param_2)
    {  ...
      {
        res = lhs.param_N < rhs.param_N;
      }
    }
  }
  return res;
}
```
- Для unordnered_MAP/SET
	
https://stackoverflow.com/questions/17016175/c-unordered-map-using-a-custom-class-type-as-the-key
	
1) В структуре/классе ключа - Переопределить operator== / operator == / operator сравнения - сравнения ключей должно учитывать, что могут быть хеш-коллизии
2) Дописать функцию хэширования (написать класс с переопределённым operator()(const Key& k), который принимает const Key& вызисляет сам хэш из данных Key-структуры и передать написанную нами функцию хеширования 3-тьим параметром в контейнер unordnered_MAP/SET (std::unordered_map<Key,std::string,KeyHasher>)
```
#include <iostream>
#include <unordered_map>
	
struct Key
{
  std::string first;
  std::string second;
  int         third;

  bool operator==(const Key &other) const
  { return (first == other.first
            && second == other.second
            && third == other.third);
  }
};

struct KeyHasher
{
  std::size_t operator()(const Key& k) const
  {
    using std::size_t;
    using std::hash;
    using std::string;

    return ((hash<string>()(k.first)
             ^ (hash<string>()(k.second) << 1)) >> 1)
             ^ (hash<int>()(k.third) << 1);
  }
};

int main()
{
std::unordered_map<Key,std::string,KeyHasher> m6 = {
    { {"John", "Doe", 12}, "example"},
    { {"Mary", "Sue", 21}, "another"}
  };
}
```
	
===================================================================

### Виртуальные методы из Конструкторов или Деструкторов

- НЕТ
Для случая создания объекта наследника, бызовется конструктор базового класса и далее начинаются проблемы:
- Т.к. конструктор Базового класса ещё не отработал полностью, а конструктор наследника вызовется только следующим за базовым, 
и таблица вирт.фций не проинициализирована полностью, то вызовется не функция наследника, вызов которой мы ожидаем, 
а вызовется функция Базового класса (если метод НЕ ЧИСТО виртуальный). В результате мы получим НЕОЖИДАЕМОЕ поведение.
- Если вызываемый метод будет ЧИСТО виртуальным (внутри конструктора) мы ещё и можем получить Исключение! **PURE VIRTUAL CALL EXCEPTION**

- Внутри Конструктора - не работает механизм виртуальных функций, потому что ещё не инициализирована таблица виртуальных функций.
- Внутри Деструктора - механизм виртуальных функций может сработать неправильно (не так как было задумано изначально), 
по причине того, что в таблице виртуальных функций будут содержаться указатели на методы уже уничтоженного класса наследника.

Подробнее: Базовый класс в нём определён конструктор с вызовом виртуальной функциии. Мы наследуемся, 
переопределяем в наследнике виртуальную функцию. При создании объекта наследника, будет сначала вызван 
конструктор Базового класса, в нём будет вызвана виртуальная фция переопределённая в наследнике, 
И ввиду того, что НАСЛЕДНИК НЕДОСФОРМИРОВАН (его конструктор ещё не отрабатывал) то как следствие возможна ошибка. 
Далее UB (undefined behavior) Вероятнее всего вызовется метод самого базового класса, если тот определён, иначе ... вопрос.

===================================================================

### Функтор это функция с состоянием
Функтор в С++ это объект у которого перегружен operator() т.е. внешне эксземпляр класса может быть похож на функцию и использоваться как функция
	
### Чем отличается копирование от присваивания?
- Копирование - инициализация объектом того же типа. Создаём новый объект "получатель" того же типа (пока пустой) и из объекта "источника" во вновь созданный копируем "содержимое источника".
- Присваивание - замена уже существующего объекта. Не создаём ничего объет "получатель" уже присуствует, и из объекта "источника" в ужеимеющийся копируем "содержимое источника".

### Какие методы в классе определяются неявно

1) Деструктор класса
2) Конструктор по умолчанию (без параметров)
3) Конструктор КОПИРОВАНИЯ. Конструктор копии (1. myclass х = у; // инициализация 2. func (х); // передача параметра 3. у = func (); // возврат из фции и как следствие получение временного
4) Оператор присваивания =
5) С++11: Перемещающий конструктор копии (он же Конструктор перемещения)
6) C++11: Перемещающий оператор присваивания =

```
(!) ПОДРОБНО:

Итак, если вы написали
class Empty {};

То, знайте, что на самом деле вы создали примерно вот такой класс:
class Empty 
{
public:
  ~Empty();  // Деструктор
  Empty();  // Конструктор без параметров
  Empty(const Empty & that);  // Копирующий конструктор по умолчанию (тупо копирует все поля)
  Empty& operator=(const Empty & that);  // Оператор присваивания
  Empty(Empty&& that) {} // C++11 // Коснтруктор перемещения (Перемещающий конструктор) 
  Empty& operator=(Empty&& that) {} // C++11 // Оператор перемещения (Перемещающий оператор присваивания)
};

//  помним так же про
  Empty * operator&();  // Оператор получения адреса
  const Empty * operator&() const;  // Оператор получения адреса константного объекта

```	

### Какова сигнатура конструктора копии / копирующего конструктора ?
	A(const A& _obj) { this = _obj; }

### Какова сигнатура оператора присваивания / оператора "равно" ?
```
A& operator=(const A& _obj) 
{
  if (&_obj == this)
    return *this;
	
  A a; a = _obj; return a;
} 
```
### Какова сигнатура перемещающего конструктора / конструктора перемещения ?
	A(A&& tmp) { ... } // см 2 вопр. ниже

### Какова сигнатура перемещающего оператора / оператора перемещения ?
	A& operator=(A&& tmp) { ... } // см 1 вопр. ниже

### Размер size_t / Какой тип у size_t
size_t = unsigned int 

### RAII подход
RAII объект (владение ресурсом передаём объекту на стеке, и этот объект уничтожает ресурс в своём деструкторе

### ГАРАНТИЯ ГАРАНТИИ ОТСУТСТВИЯ ИСКЛЮЧЕНИЙ БЕЗОПАСНОСТИ ИСКЛЮЧЕНИЙ
- Гарантия отсутствия исключений - noexcept - ни при каких обстоятельствах функция не генерирует исключ, а если генерирует, то они внутри неё же и отлавливаются
- Базовая гарантия безопасности исключений - если возникло исключение, то все объекты остаются в согласованном состоянии, утечек ресурсов не произошло
- Строгая гарантия безопасности исключений - если во время операции возникло исключение, то программа останется в том же состоянии, которое было до начала выполнения операции. (ДРУГИМИ СЛОВАМИ если при выполнении операции возникает исключение, то это не должно оказать какого-либо влияния на состояние приложения )

Проблемный код с потенциальным исключением, которое приводит к Утечке памяти
```
template<class T> struct Arr
{
 void resize(size_t n) {
   // *1 // new - std::bad_alloc - при нехватке памяти
   // НО мы ещё не успели изменить данные объекта Arr, и фактически Arr 
   // останется таким же как до запуска операции resize
   // *2 // У типа Т потенциально может быть конструктор, который
   // может бросить исключение, даже если его не бросит само new
   // (!) НО ЕСЛИ исключение происходит в конструкторе внутри оператора new
   // то компилятор гарантирует нам, что память будет освобождена
   // останется таким же как до запуска операции resize
   T* ndata = new T[n]; // *1 *2

   // *3 // оператор= присваивания тоже может бросить исключение,
   // и без try-catch и тогда для указателя ndata не вызовется delete и будет
   // УТЕЧКА ПАМЯТИ
   for(size_t i = 0; i!= n && i != size_; ++i)
     ndata[i] = data_[i]; // *3
	
   delete [] data_;
   data_ = ndata;
   size_ = n;
 }
 
 T* data_;
 size_t size_;
};

```

РЕШЕНИЕ 1
```
template<class T> struct Arr
{
 void resize(size_t n) {
   // *1 оставит всё согласованным и *2 память очистится компилятором
   T* ndata = new T[n];

   // *3 // оператор= присваивания может бросить исключение, и *ndata утечёт
   try {
     for(size_t i = 0; i!= n && i != size_; ++i)
       ndata[i] = data_[i]; // *3
   } catch (...) {
     delete [] data_;
	 throw;
   }
	
   delete [] data_;
   data_ = ndata;
   size_ = n;
 }
 
 T* data_;
 size_t size_;
};
```

РЕШЕНИЕ 2 - RAII объект (владение ресурсом передаём объекту на стеке, и этот объект уничтожает ресурс в своём деструкторе)
```
template<class T> struct Arr
{
 void resize(size_t n) {
   unique_ptr<T[]> ndata(new T[n]);

   // *1 // ЕСЛИ оператор= присваивания бросит исключение,
   // то начнётся размотка стека, и 
   // деструктор unique_ptr освободит указатель ndata
   for(size_t i = 0; i!= n && i != size_; ++i)
     ndata[i] = data_[i]; // *1
	
   data_ = std::move(ndata);
   size_ = n;
 }
 
 unique_ptr<T[]> data_;
 size_t size_;
};
```

РЕШЕНИЕ 3 - Проделать все опасные операции над копией объекта и если что-то пошло не так просто оставить копию и забить на выполнение операции




### Разница между abort() и exception()
- abort() -> сразу terminate();
- exception() -> если оно не влетело в try{}catch(){} блок, то по сути это тот же самый abort()
- (!) abort() - Деструкторы для атоматических переменных, для abort() не вызовутся.
- (!) exception() - Деструкторы для атоматических переменных, начинают вызываться ТОЛЬКО, ТОГДА, КОГДА мы попали в catch-БЛОК и вот тогда начинается stack-unwinding Раскрутка стека.

### Что происходит, когда мы делаем fork()
- Форкаем процесс, и с памятью процесса происходит следующее. Мы знаем, что создаётся копия процесса, НО происходит это всё по технологии COW (Copy On Write), то есть если кто-то начинает менять какую-то страницу памяти, то только тогда она реально начинает копироваться.

### Свой класс строки
- иммутабельный (будем редко их менять) - можно на основе std::array
- достаточно часто изменяем строки, но операции вставки и удаления чаще всего в конец, тогда сделаем на основе std::vector
- достаточно часто изменяем строки, но операции вставки и удаления чаще всего в середину, тогда сделаем на основе std::list (возможно std::deque)
- copy on write строки

### CString - Copy-On-Write (COW)
- Технология позволяет эффективнее работать с временными объектами (ибо мы не копируем объект сразу, а лишь тогда когда объект изменится)
- (создали копию, ориг=буфер, ориг_и_копия шарят буфер, как только кто-то буфер начинает меняться, копия РЕАЛЬНО скопирует содержимое буфера перед его изменением к себе) При копировании на самом деле ничего не копируется по началу, копия и оригинал просто указывают в одну и ту же память, как только кто-то решит внести изменение в оригинал или в копию, вот тогда, копия РЕАЛЬНО выполнит копирование и перестанет ссылаться на оригинал.
- По технологии Copy-On-Write вместо того, чтобы строка уникально владела динамическим объектом, который создан в куче, можно сделать так, чтобы несколько строк одновременно ссылались на динамический объект. Тогда в этом динамическом объекте будет счетчик ссылок — сколько одновременно строчек работает с ним. Код будет выглядеть так:
- Только старый Windows CString класс последним использовал COW-технологию 
- Начиная с C++11 COW строчки запрещены в стандарте. 
Подробнее тут: https://habr.com/ru/company/oleg-bunin/blog/352280/

```
class string_impl {
  char* data;
  size_t size;
  size_t capacity;
  size_t use_count; // <===
  // ...
};
```

### std::string - Работа с маленькими строками SSO - Short String Optimsation
- Если строка маленькая, ТО она в HEAP не сохраняется, не делается лишняя аллокация памяти, короткая строка (пустая, или 22 символа) будет хранится внутри спецального union-а, в котором есть специальный массив small_buffer_t { char data[sizeof(no_small_buffer_t)]; }
- строка нулевой длинны не будет храниться в куче и не будут динамически аллоцироваться
Подробнее тут: https://habr.com/ru/company/oleg-bunin/blog/352280/

```
class string {
  size_t capacity;
  union {
    struct no_small_buffer_t {
      char* data;
      size_t size;
    } no_small_buffer;
    struct small_buffer_t {
      char data[sizeof(no_small_buffer_t)];
    }small_buffer;
  }impl;
}
```



====================================================	
	
# МНОГОПОТОЧНОСТЬ

### Что будет, если std::thread упадёт ДО join()
- Вызовется terminate() и программа упадёт

### Что будет, если создать std::thread t; и не написать t.join();
### Что будет, если создать std::thread t; и не написать t.detach();
- Поток который завршит своё выполнение, программа упадёт ибо в деструкторе потока вызовется terminate()

## Проблемы многопоточности

### 1) гонки потоков
— задержки(delay) / Слипы / таймауты (плохо, но на крайняк можно)
— использовать атомики (атомарные переменные и атомарные операции)

### Сколько нужно мьютексов, чтобы случился дедлок?
Ответ: МИНИМУМ ДВА

http://www.cyberforum.ru/cpp-beginners/thread921902.html
	
### 2) Дедлоки (avoiding circular wait)
- решается мьютексами
    -- mutex - базовый мьютекс - mutex.lock() | mutex.unlock()
    -- timed_mutex - мьютекс с тайм-аутом 
    -- recursive_mutex - мьютекс, который может быть заблокирован рекурсивно из того же потока 
    -- recursive_timed_mutex - мьютекс с таймаутом, который может быть заблокирован рекурсивно из того же потока 
    -- shared_mutex - provides shared mutual exclusion facility 
    -- shared_timed_mutex - provides shared mutual exclusion facility 
- порядком обращения к месту, где случается деллок (правильный порядок захвата мьютексов и их отпускания)
- std::lock_guard, std::unique_lock, std::try_to_lock, std::lock на мьютексах
    -- lock_guard - реализует обёртку владения мьютексом строго в области видимости 
    -- unique_lock - реализует подвижного мьютекс собственности обертку 
- condition_variable - решается условными переменными 
    -- notify_all_at_thread_exit
- семафоры как частный случай мьютекса

- А так же для асинхронных уже не проблем а просто решений в 11 стандарте так же есть:
- packaged_task()
- future - future_error
- promise - future_error
- async
- launch

И ЕЩЁ общение между потоками / синхронизация потоков:
- События - std::condition_varianle
- Мьютекс - std::mutex
- Семафоры
- Критические секции (аналог в std std::lock_guard<std::recursive_mutex> locker(m);)
- SpinLock 
- Атомарные операции 

	defer_lock_t  -  try_to_lock_t  -  adopt_lock_t 
	
===================================================================

### explicit
explicit - когда мы хотим запретить НЕЯВНОЕ преобразование типов для параметра конструктора

### Этапы сборки
Этапы сборки: - проект в исполныемый файл
- директивы препроцессора все убираются, на их места ставтися всё что нужно, весь код формируется в один большой кусок кода
- этап компиляции - компиляция CPP-фапйлов файлов в объектные файлы
- линковка - нужна для сопряжения / совмещения в один общий исполняемый файл, для объединения нескольких объектных файлов в один исполняемый

Почему аргументы макросов нужно брать в скобки - чтобы по ходу использования макроса

Как headers - 

### ПОЛИМОРФИЗМ
1. Динамический - на этапе выполнения 
- механизм виртуальных функций (оверрайдим методы в классах наследниках и получаем по итогу разное поведение)
2. Статический: - на этапе компиляции
- 2.1 Шаблонные методы считаются стат.полим.
- 2.2 Разная сигнатура (2 метода с одинаковыми именами имеют разную сигнатуру по параметрам)

### Что произойдёт, если:
```
delete nullptr;
delete NULL; (макрос)
delete 0;
```
- НИЧЕГО не произойдёт.
- nullptr КАСТИТСЯ к bool - значению, и не кастится к арифметическому значению.
- только в этом плане nullptr это более безопасное зануление указателя, чем приравнивание такового к числу ноль / нулевому адресу

### Виртуальный деструктор - A базовый, B - наследник от А
```
class A
class B : public A
B b;         // A() // B() // ~B() // ~A()
A* a = new B; 
delete a;    // A() // B() //         ~A()
=>
virtual ~A()
delete a;    // A() // B() // ~B() // ~A()
```
- деструткор вызовется только для А, и не вызовется для B
- Утечка памяти, чтобы она не происходила, поэтому деструктор А делаем вритуальным, чтобы вызывался деструктор для ~B, и потом для ~A 

### Как вызовутся конструкторы классов, полей, наследников?
```
struct C : ABase, BBase
{
  BField b;
  AFirld a;
}
```
- конструкторы ABase и BBase вызовутся ранее конструкторов полей "а" и "b"
- конструктор вызовется у полей b - он раньше объявлен, a - он позже;
- сначала деструкторы Сам класс, далее сначала поле a, потом поле b и только потом родительсвие опять же в обратном порядке ~BBase ~ABase

### Можно ли бросать Исключение в КОНструкторе / Исключения в КОНструкторе
- Exception в конструкторе, он обработан в программе, но не обработан в рамках конструтора
могут произойти утечки, ибо для неких полей, которые являются указателями, если для них выделится память, 
то после исключения для них могут не вызваться из delete - в случае, если new - были в конструкторе, а delete были в деструкторе.
Но так как конструктор до конца не отработал, и деструктор не будет соотвественно для класса вызван.
ЗАЩИТА и РЕШЕНИЕ: УМНЫЕ УКАЗАТЕЛИ - даже в таких случаях деструкторы будут вызваны для созданных объектов.
```
struct A {
  A() {
    pb = new B; // pb - утечёт
    trow 1;
  }
  ~A() {
    delete pb; 
  }
  B *pb;
}; // Для данного класса т.к. есть указатель
// потребуется дописать и констр.копии и оператор=
// и по хорошему перемещающие-констр+операот=
// чтобы дописать всю доп работу с указателем B *pb

void test () {
  try { A a; }
  catch(...) {
  // сам pb занулится, а вот память под объект new B, останется в памяти
  }
}
```
	
### Можно ли бросать Исключение в ДЕСтрукторе / Исключения в ДЕСтрукторе
```
struct A {
  ~A() // можем добавить noexcept(false) и тогда можно бросать исключение
  { trow 1; }
};
try{
  A a;
}
catch(...) { }
// Программа упадёт: // 1. Все деструкторы по умолчанию помечены как noexcept, допишем к деструткору noexcept(false) и тогда можно бросать исключение и падения программы не будет
```

### Опасно ли бросать исключение в ДЕСтрукторе / Исключения в ДЕСтрукторе
```
struct A {
  ~A() // можем добавить noexcept(false) и тогда можно бросать исключение
  { trow 1; }
};
try{
  A a;     // 2. сработает деструктор и выбросится второе исключение и это Необрабатываемая корректно ситуация, программа упадёт
  throw 2; // 1. Раскрутка стека пошла, начинают вызываться детсрукторы всех локальный переменных в данной области видимости
}
catch(...) { }

// сначала throw 1;   // 1. Раскрутка стека пошла, начинают вызываться детсрукторы всех локальный переменных в данной области видимости
// потом A a; -> ~A() // 2. сработает деструктор и выбросится второе исключение, по стандарту, когда на этапе размотки стека происходит ещё одно исключение, то вызовется команда std::terminate(), и программа упадёт
```

### POD - plain old data тип
```
class CD {
  CD() = default;
};
class CD1 {
  CD1() { }
};

CD  - может расцениваться как POD - plain old data тип
CD1 - НЕ может расцениваться как POD - plain old data тип
```

### Размер пустого класса
```
struct A{};
struct B : A {};
sizeof(A) = 1, sizeof(B) = 1
// ОТВЕТ Empty Base class optimization - если базовый пустой и пустой наследник, то оба будут весить 1 байт
```

**Правило трёх 3-ёх**: Если у нас есть нетривиальный конструктор, либо оператор присванивания либо констр.копии, 
и ЕСЛИ нам нужно реализовать один из этих конструкторов, то нам нужно реализовать сразу ВСЕ.

**Правило пяти 5-ти**: для МУВ-семантики: Если у нас есть нетривиальный конструктор, либо оператор присванивания либо констр.копии / оператор перемещения, констр.перемещения, 
и ЕСЛИ нам нужно реализовать один из этих конструкторов, то нам нужно реализовать сразу ВСЕ.


### placement new - Как можно создать объект класса по определённому адресу в памяти ? по конкретному адресу в памяти ?
Использовать оператор placement new

ЗАЧЕМ:
- Однин и тот же участок памяти чтобы брать и с ним работать. Чтобы лишний раз не "дёргать" кучу.
- Если у нас памяти мало, или есть выделенный участок
- Если мы хотим облегчить нашу работу не обращаться к ОС для удаления или для создания, просто можем брать и создавать объкты и удалять их по одним и тем же указателям

### Как запретить наследование от класса:
- Заприватить конструкторы 1. обычный 2. костр.КОПИИ 3. оператор присваивания
- final - ТРЕБУЕТ УТОЧНЕНИЯ !!! - и нельзя будет наследоваться от этого класса ... 
```
С С++11
class Foo final {};
class Bar: public Foo {}; // Fails to compile as Foo is marked as final
```
	
### Умные указатели / smart pointers / smart_pointers
- uniqie_ptr - он сам уникально владеет объектом (когда наоборот хотим не шарить объект а гарантировать монопольное использование)
- shared_ptr - умный указатель с счётчиком ссылок на объект (объект жив пока счётчик ссылок не равен нулю)
- weak_ptr - он хранит указатель так же как и shared_ptr, но он не считает ссылки на объекты, нужен для решения проблемы циклических/колцевых ссылок (2 шаред поинтера не могут удалиться по итогу)

### shared_ptr() и make_shared()
```
void func() {
  func1(std::shared_ptr<A>(new A), func_ex());
}
```

1) **shared_ptr** - состоит из ДВУХ вещей: 1. Сами данные и 2. Контрольный блок с счётчиком ссылок
- Память под shared_ptr и указатель на сам объект выделяется за два шага ВЫДЕЛЕНИЯ ПАМЯТИ и фрагментировано, то есть в разных местах.
- В случае с shared_ptr(new type) память для счетчиков аллоцируется отдельно от объекта. И поэтому, располагаться эти счётчики могут где угодно (почти наверняка - далеко от расположения объекта)
- МНУС (*) Если вы не используете функцию make_shared, то придется использовать явное выражение **new** для создания объекта, прежде чем передавать его в конструктор shared_ptr, а **new** может бросить исключение (std::bad_alloc)

2) make_shared() выделение КАК-ПРАВИЛО (typically performs only one allocation) произойдёт за одно выделение памяти, согласно стандарту.
- 1. инициализацию shared_pointer-a и 2. самих данных на которые указывать будет поинтер ЗА ОДНО ВЫДЕЛЕНИЕ ПАМЯТИ.
- В случае с make_shared счетчики объектов всегда располагаются рядышком с самим объектом. Потому что расположены в том же самом блоке данных, в котором расположен сам объект.
- ПЛЮС (*) Используйте функцию make_shared для создания указателя shared_ptr. Функция make_shared безопасна в отношении исключений. Для выделения памяти под блок управления и ресурс используется один вызов, что снижает накладные расходы. 

3) СУТЬ ПРОБЛЕМЫ
Создаём std::shared_ptr из свежесозданного сырого указателя может приводить к утечке памяти:
```
func ( std::shared_ptr<Bar>(new Bar), foo() );
```
- Для вычисления аргументов функции func(...) необходимо вызвать: 1. new Bar; 2. конструктор std::shared_ptr; 3. foo().
- Компилятор может **перемешать порядок выполнения эти трёх операций**, ибо порядок обработки параметров функции стандартом не определён, итог:
1. new Bar; 2. foo(); 3. конструктор std::shared_ptr. Если при этом **в foo() возникнет исключение** — получаем утечку экземпляра Bar.
- Ни один из следующих примеров кода не содержит потенциальную утечку (но мы ещё вернёмся к этому вопросу):
```
auto bar = std::shared_ptr<Bar>(new Bar);

auto bar = std::shared_ptr<Bar>(new Bar);
process(bar, foo());

process(std::shared_ptr<Bar>(new Bar));
```

4) РЕШЕНИЕ std::make_shared бесполезный

Начиная с c++17 утечка памяти в том хитром редком примере, ради которого в STL был добавлен std::make_shared, уже невозможна.

### Q: Объясните принцип полиморфизма. 
Полиморфизм - возможность работать с разными типами одинаковым образом. Статический - перегрузка функций, шаблоны. Динамический - виртуальные функции.

### Q: Сложность алгоритма
- Сложность алгоритма - в общем случае, это количественная характеристика, которая говорит о том, сколько времени, либо какой объём памяти потребуется для выполнения алгоритма.
- Измеряется сложность алгоритмов в элементарных шагах - то, сколько действий необходимо совершить для его выполнения. Любой алгоритм включает в себя определённое количество шагов и не важно на каком устройстве он будет запущен, количество шагов останется неизменным. Эту идею принято представлять в виде Big O (или О-нотации).
- Big O показывает то, как сложность алгоритма растёт с увеличением входных данных. При этом она всегда показывает худший вариант развития событий - верхнюю границу.

### Q: Двоичный поиск / Бинарный поиск / сложность бинарного поиска
- Особенность: Бинарный поиск можно осуществлять ТОЛЬКО внутри отсортированного массива
- Сложность O(log(n))

### Разница между оператором [] и оператором at() в векторе vector
- vector.at(i); Делает попытку обратиться к элементу и  Бросает исключение std::out_of_range  при выходе за пределы вектора 
- operator[] делает попытку считать данные и падает без бросания исключения


//
// ПОМНИМ ЗАЧЕМ ВООБЩЕ НУЖНО СЛОВО friend
//

friend - функции - это функции, которые не являются членами класса, однако имеют доступ к его закрытым членам - переменным и функциям, которые имеют спецификатор 
private.

friend - классы - Все методы класса A будут иметь доступ к скрытым членам класса B.
```
class A { };
class B
{ friend class A; };
```

```
class B {
  public: 
  virtual ~B() = default;
};
class Child : protected B
{
  friend void f(); // этой строки не было. БОЛЕЕ ТОГО именно добавив friend код скомпилируется
};

void f() {
  B* b = new Child;
}
```


# explicit - зачем нужно
Чтобы не позволить вызвать конструктор от одного аргумента неявно

### [Q] 48. Как запретить неявное преобразование типа, выполняемое конструктором инициализации?
Ответ: Используя ключевое слово explicit (ПРИМЕР см. 2 вопросами ниже)

### [Q] 59. Для чего объявлять конструктор с ключевым словом explicit?
Таким образом происходит подавление неявное приведение типа. Такой конструктор можно вызывать только явно. (ПРИМЕР см. вопросом ниже)

### [Q] 50. Для чего служит ключевое слово explicit?
Это модификатор для запрета неявного преобразования типов объектов аргументов конструктора.

```
#include <iostream>
class B
{
  int m_var;
  public:
  B() { }
  B(int var) : m_var(var) { }
  void print() { std::cout << "print!" << std::endl}
}

void fun(B b)
{ b.ptint(); }

int main()
{
  B obj1(10);  // Обычный вызов конструктора
  B obj2 = 10; // implicit call 
               // НЕЯВНОЕ ПРЕОБРАЗОВАНИЕ и вызов Конструктора с параметром
               // На самом деле вызовется  obj2( 10 );
  fun(obj1); // Обычный вызов конструктора
  fun(30);   // implicit call 
             // НЕЯВНОЕ ПРЕОБРАЗОВАНИЕ и вызов Конструктора с параметром
             // На самом деле вызовется  fun(B( 30 );
}
```
ЕСЛИ в классе перепишем так: **explicit B(int var) : m_var(var) { }**

ERROR 1 будет: B obj2 = 10; // convertion from int to NON Scalar type

ERROR 2 будет:   fun(30);   // can not convert from int to B


### [Q] Назовите четыре варианта приведения типов?

1) const_cast – аннулирует const и volatile -  убирает константность объекта. Никакие другие приведения в С++ стиле не позволяют такое сделать.
2) static_cast – иерархическое преобразование с проверкой на этапе компиляции - явное приведение типов, например, и в основном 1) для встроенных типов (Int к double), то будут использованы встроенные в C++ правила их приведения. 2) Если это типы, определенные программистом, то будут использованы правила приведения, определенные программистом.
static_cast - ещё варианты: приведение неконстантных объектов к константным. между указателями корректен, только если один из указателей - это указатель на void или если это приведение между объектами классов, указателей на базовый класс к указателю на производный.
3) dynamic_cast — проверяется на этапе выполнения RTTI -  применяется для выполнения безопасного понижающего приведения. Позволяет определить, принадлежит ли объект данного типа некоторой иерархии наследования. Единственное приведение, которое не может быть выполнено с использованием старого синтаксиса в Си-стиле. Требует определённых временных затрат.
4) reinterpret_cast — несвязанное преобразование с проверкой на этапе компиляции -  Обычно используется, чтобы привести указатель к указателю, указатель к целому, целое к указателю. Умеет также работать со ссылками. Результат может быть некорректным, никаких проверок не делается. Считается, что вы лучше компилятора знаете как на самом деле обстоят дела, а он тихо подчиняется. Не может быть приведено одно значение к другому значению. 

Предназначен для низкоуровневых приведений, которые порождают зависимые от реализации (непереносимые) результаты, например приведение указателя к int. Вне низкоуровневого кода такое приведение должно использоваться редко.


### 10.2 Почему стоит использовать преобразования типов в C++ стиле?
Потому что оно более безопасное. Си-стиль позволяет привести почти всё что угодно ко всему, без проверок от компилятора! C++-стиль приведения более заметен, чем Си-стиль.


### 11.1 Чем отличается dynamic_cast от других вариантов приведения типов?
Безопасное приведение по иерархии наследования, в том числе и для виртуального наследования.

dynamic_cast<derv_class *>(base_class_ptr_expr)

Используется RTTI (Runtime Type Information), чтобы привести один указатель на объект класса к другому указателю на объект класса. 
Классы должны быть полиморфными, то есть в базовом классе должна быть хотя бы одна виртуальная функция. Если эти условие не соблюдено, ошибка возникнет на этапе компиляции. Если приведение невозможно, то об этом станет ясно только на этапе выполнения программы и будет возвращен NULL.

	
# Различие между С и С++ / между C и C++ / Разница между С и С++ / между C и C++

- Си (структурное прогр.) | В С++ (Объектно ориентированное прогр.)
- Си (отсуствует полиморфизм) | В С++ (Полиморфизм работает через виртуальные функции)
- Си (можно вызывать main() из других фций) | В С++ (нельзя вызывать main() из других фций)
- В С код программы хранится в файлах с раширением ".c" | В С++ код программы хранится в файлах с раширением ".cpp"
- В С для работы с консолью используются scanf() и printf() | В С++  std::cout и std::cin
- default-аргумент (В Си у фций НЕТ аргументов по умолчанию | В С++ у фций МОЖНО использовать аргументы по умолчанию)
- Объекты (В Си нет модификаторов доступа private, protected, есть структуры в которых всё public | В C++ структуры могут содержать в себе все три мдификатора доступа public, private, protected, и в классах тоже)
- Структура / Класс (1. в структуре по умолчанию все поля публичные, в Классе по умолчанию все поля приватные. 2. В структуре по умолчанию публичноре наследование, в классе по умолчанию приватное наследование)
- Наследование (То, что в C++ - наследование, в C - это просто структура в структуре. При программировании в стиле C++ применяются такие красивые и звучные слова, как "класс Circle порожден от класса Point" или "класс Point наследуется от класса Circle и является производным от него". На практике все это словоблудие заключается в том, что структура Point - это первое поле структуры Circle.
При этом реальных усовершенствований два. ПЕРВОЕ - поля Point считаются так же и полями Circle, в результате доступ к ним записывается короче, чем в C. ВТОРОЕ - в обоих структурах можно иметь функции-методы, у которых имена совпадают с точностью до имени структуры. Например, Point::paint и Circle::paint . Следствие - не надо изобретать имена вроде Point_paint и Circle_paint, как это было в C,)
- new и delete (В Си пользоватлись malloc() и free | В С++ пояились new и delete) // Point *p =malloc(sizeof(Point)); и free(p);  Point *p = new Point; delete p;)
- Конструкторы и деструкторы (В Си заведя переменные их надо проинициализировать отдельно, автоматом конструктор не отработает и не поможет нам, и деструктора тоже нет, который очистит за нас содержимое перемиенных и указателей в структуре | В С++ конструктор вызывается автоматически, но инифилизацию можно написать сразу в него, Деструктор есть вызовется атоматом, деинициализирует сам содержимое POD элементарных типов, содержимое указатели праввда надо всё равно осводить в ручную)
- virtual и firiend - Виртуальные функции - (В Си отсуствует механизм виртуальных фций, поэтому механизм работы с таблицами виртуальных функций (через указатель на соотвествующую таблицу для структуры), если необходимо, то придётся реализовать "своими руками". | В C++ есть поддержка ключевого слова virtual и механизма виртуальных функций и работа с указателем на vtable, а так же есть и ключевое слово friend)
- Исключения (В Си НЕТ поддержки механизма обработки исключений | В С++ ЕСТЬ поддержки механизма обработки исключений)
- Перегруженные операторы (В Си НЕТ перегрузки операторов | В С++ ЕСТЬ перегрузка операторов)
- Ссылка (В Си НЕТ ссылок, но есть указатели | В С++ ЕСТЬ и ссылки и указатели)
- inline (В Си НЕТ поддержки inline фций | В С++ ЕСТЬ поддержки inline фций)
- namespace (В Си НЕТ поддержки ключевого слова namespace | В С++ ЕСТЬ поддержка namespace для непересекаемости имён объектов)
- template 

	
	


АССОЦ-КОНТЕЙНЕР:
- оператор меньше
- 

===================================================================

	
# WINDOWS

	
### Типы ОБЪЕКТ ЯДРА
- маркеры доступа / access token
- события / event
- файлы / file
- проекции файлов / file mapping
- порты завершения ввода-вывода / I/O completion port
- задания / Job
- почтовые ящики / mailslot
- мъютексы / mutex
- каналы / pipe
- процессы / process
- семафоры / semaphore
- потоки / thread
- ожидаемые таймеры / waitable timer

### ОБЪЕКТ ЯДРА
(ОЯ) ОБЪЕКТ ЯДРА - Это Блок памяти внутри Ядра, выделенный под оперделённые задачи, которые мы просим выполнить ядро, или которые ядру нужно сделать самому
- HANDLE объекта (дескриптор из таблиц дескрипторов в процессах, "указывающих" на сами объекты ядра, которые находятся в области памяти ядра)
- имя объекта ядра
- Класс (ID) защиты объекта ядра
- счетчик количества пользователей объекта ядра
- обладают свойством наследования
- Создаются через спец.функции HANDLE CreateSemaphore / HANDLE CreateFile / 
- Закрываются обязательно через HANDLE объекта и спец.функцию BOOL CloseHandle( HANDLE hObject ); // принимает дескриптор объекта

### HANDLE
HANDLE - 32разрядный INT - Дескриптор какого-то объекта Ядра. У каждого процесса есть своя таблица Дескрипторов, в этих таблицах и хранятся хэндлы  "указывающие" на сами объекты ядра, которые находятся в области памяти ядра.

### ПРОЦЕСС
ПРОЦЕСС - 1. эксземпляр исполняемой программы со своим отдельно под неё выделенным виртуальным адресным пространстовм 2. некое состояние выполнения операции
- 0. Хранит в себе 1. PID - уникальный идентификатор процесса 2. список открытых дескрипторов (HANDLE) различных системных ресурсов 3. статистическую информацию.
- 1. собственное виртуальное адресное пространство (Адресное пространство - диапазон адресов виртуальной памяти, которым может пользоваться процесс)
- 2. исполняемый код / исполняемая программа
- 3. данные, потребляющие ресурсы ОС, то есть данные проецируемые на виртуальное адресное пространство
- 4. СТЕК программы
- 5. Имеет связанный набор РЕГИСТРОВ в которых хранятся состояния СЧЁТЧИКОВ (Пример: "Счётчик команд" - регистр с адресами следующей стоящей в очереди на выполнение команды)
	
### ПОТОК
ПОТОК - Поток (thread) – независимый поток исполнения команд в ранках процесса. Поток не процесс! POSIX определяет набор интерфейсов (функций заголовочных файлов) для программирования потоков.
- Процесс в ОС может состоять из нескольких потоков, выполняющихся "параллельно"
- Существует как минимум один основной поток / главный поток в рамках выполняющегося процесса.
- Все потоки процесса выполняются в адресном пространстве процесса.
- Все потоки процесса пользуются ресурсами процесса.
- Он отвечает за ИСПОЛНЕНИЕ кода в адресном пространстве процесса
- Имеет набор регистров процессора, отражающих состояние процессора
- Имеет 2 стека: 1-ый Юзер-модный, а 2-ой Кернел-модный. Для исполнения в 1. юзер-мод (пользоват.) режиме, а 2ой для исполнения в 2. кернел-мод режиме (режиме ядра)
- Стек потока так же содержит параметры всех функций и локальные переменные для выполнения потоком кода 
- И так же содержит в себе TLS ThreadLocalStorage - локальная память потока, где переопределены, в частности, все глобальные константы, к-рые ф-ции CRT-библиотеки юзают. Например, такая как errno. Именно TLS - гарантия, что ф-ции CRT-библиотеки будут работать корректно.
- Поток завершает свою работу либо сам, либо принудительно, если завершает работу его Процесс-родитель.

### Createthread разница _beginthreadex

ТЕЗИС: Лучше использовать _beginthreadex, чем CreateThread().
	
- _beginthread - актуален, если используешь многопоточную версию crt. Иначе возникнут проблемы при использовании глобальных переменных и т.п., ибо в первой версии crt и понятия не было о многозадачности (см.книгу Джефри Рихтера), а “ex” – просто более широкий набор параметров, и всё. 
- _beginthread и _beginthreadex - функция библиотеки CRT (си рантайм либ), НЕ кроссплатформенна, но приспособленна для работы с С, т.е. она делает дополнительные манипуляции, что бы стандартные библиотеки корректно работатли (+ вызывает CreateThread()) .
- _beginthreadex - RunTime-функция: она запустится - выделит память в Thread Local Storage и будет ею пользоваться.
- Когда процесс будет завершать работу, то _beginthread(ex) - удалит так же ещё и вспомогательную информацию, что хранится в этих самых рантайм-бибоиотечных объектах.
- (!) Если объекты рантайм-библиотечные не удалить, то может произойти своеобразная утечка памяти, от чего ЗАЩИЩАЕТ _beginthreadex.

- CreateThread - чисто Win32Api'шная функция, просто создающая на уровне ядра объект потока.
- CreateThread  (не создаёт такого окружения для работы с рантаймовскими библами и объектами, как _beginthread(ex)) не запускается таким образом, чтобы создать рантайм-объекты и как следствие он не вызывает и деструкторов этих объектов. 
- (!) Если объекты рантайм-библиотечные не удалить, то может произойти своеобразная утечка памяти, от чего НЕ защитит CreateThread().

- СУТЬ: 
http://forum.vingrad.ru/forum/topic-47554.html

В общем если пишешь на С, то лучше пользоваться вторым.
Рихтер в статье CreateThread vs. _beginthread очень подробно описывает почему _beginthread предпочтительнее. Она создает **Thread Local Storage (TLS)**, где переопределены, в частности, все глобальные константы, к-рые ф-ции CRT-библиотеки юзают. Например, такая как errno. Именно TLS - гарантия, что ф-ции CRT-библиотеки будут работать корректно. 

The C runtime library was delivered in a UNIX context, in which there is no distinction between processes and threads. In the Windows context, many threads can be executing in a single address space. 

Microsoft has provided an alternative function to CreateThread, called _beginthreadex, to be used with the programs that use multiple threads at the same time they use the C runtime library. The problem occurs with any globally accessible variable used by this library ( there are several of them ). The Microsoft solution is to have the C runtime library provide a copy of each of these variables for each thread. Then, when a thread interacts with the runtime library, variables are shared only between the runtime code and the thread, not among all threads. The _beginthreadex function creates the copy for a thread in conjunction with an embedded call to CreateThread.

### СООБЩЕНИЯ в WINDOWS
- Внешние устройства (клава, мышь)
- Уведомления ПО о событиях в ОС от ОС
    - ПО_1 -- уведомить_ОС --> ОС -- сообщение_ПО --> ПО_2
    - ПО_1 --> ПО_1 (самому себе)
    - ПО_1 --> ПО_2 --> ПО_3
```
      Сообщения от
 ---- драйверов
 |    Мыши клавы
 |
 |__> Кладётся в 
 ---- очередь сообщений
 |
 |    Далее смотрим
 |__> С каким активным окном
 ---- связанно сообщение
 |
 |__> Сообщение
         \/
   т---------------т
   | Приложение    |
   |  _____V_____  |
   |  |  оконная | |
   |  |  функция | |
   |  |____v_____| |
   |  | Обработка| |
   |  | сообщения| |
   |  |__________| |
   |_______________|
   
```

### Оконная функция HRESULT CALLBACK WndProc(wnd, IntMsg. WPar, LPar)
Доступна для вызовов извне приложения только ОС

Функция окна - формирует сообщения, описывающие события м направляет его в нужную функцию окна.
- ф-ция анализирует сообщения и выполняет соотвествующие действия
- обрабатывает сообщения, распределяемые окну
- ОДНО Окно <=> ОДНА Оконная функция
- вызов в функции окна осуществляет только ОС, из main() её не вызвать
```
Окна:
- Родитель
  - потомок_1
  - потомок_2
  ...
  - потомок_N
```
Главынй родитель - Окно DESKTOP (дочерние окна без родителя)
	
```
  _______
 |над    |________________
 |потомок|   ПОДРОДИТЕЛЬ  |
 |_______|                | 
   |             _______  |
   |            |над    | |
   |            |потомок| |
   |            |_______| |
   |______________________|
	
```	
Зкрыли родителя, Закрылм всех потомков
	
### PostMessage SendMerssage
- SendMessage - Вызывает оконную функцию напрямую, БЕЗ постановки в очередь обработки сообщений
- PostMessage - ставится в очередь обработки сообщений внутри программы
- GetMessage - получить сообщение из очереди сообщений
- TranslateMessage - трансляция, например от клаиватуры
- DispatchMessage - перенаправление оконной функции (return результат обработки)
	
### FileMapping
```
|Файл|Проекция Файла|Окно проекции (View)|
```	
- CreateFileMapping 
- MapViewOfFile 
- lpMapAddress + iViewDelta
- UnmapViewOfFile
- CloseHandle
	
### Wait функции
- WaitForSingleObject
- WaitForMultipleObjects

### Синхронизация потоков
- События
- Мьютексы
- Семафоры
- Крит. секции

### Синхронизация процессов
- Именовынные Мьютексы
- Condition Variable
- разделяемая память / ФайлМэппинги
- Оконные сообщения
- Пайпы, Сокеты
- МэйлСлоты

### Оконная функция
http://lavimidj.ru/860/

- SendMessage (синхр.)
- PostMessage (асинхроння)
- GetMessage (while ((iGetOk = GetMessage(&msg, NULL, 0, 0 )) != 0) //цикл сообщений)
- TranslateMessage
- DispatchMessage
- PostQuitMessage
- WndProc - Обработка сообщений
	
### COM-объекты и QueryInterface
У COM технологии есть всего 2 отвественности:
- Запросить интерфейс
- вести подсчёт ссылок
	
```
	                      VTBL
	   COM-Component      ptr to Interface_1
Client --> VTBL-Pointer --->  ptr to Interface_2  
	   Obj. Data          ptr to Interface_N
```
	
### COM-объекты и QueryInterface
- Клиент использует QueryInterface-метод чтобы запросить интерфейс, содержащийся в COM-компоненте
```
COM-комп.1 | COM-комп.2
____       | ____
|   -- A   | |   -- X
|   -- B   | |   -- Y
|__ -- C   | |__ -- Z

A,B,...Y,Z - Интерфейсы
```
- у любого COM интерфейса/компонента/класса есть GUID
- COM компонент хочет воспользоваться интерфейсом, и вызывает AddRef()
- COM компоненту больше не нужен интерфейс - вызываем Release()
- COM компонент остаётся загруженный в память пока счётчик ссылок на него не станет равен нулю
- CLSID - используется в COM-библиотеках, чтобы загрузить СОМ-компонент

### Соглашения о вызовах _cdecl, _pascal, _stdcall / Windows
- _pascal - ПРЯМОЙ - параметры передаются на стек в ПРЯМОМ порядке
- _cdecl - ОБРАТНЫЙ - параметры передаются на стек в ОБРАТОМ порядке
- _stdcall / Windows - ОБРАТНЫЙ - параметры передаются на стек в ОБРАТОМ порядке + Вызванная продпрограмма освобождает стек!

### Статический импорт библиотек DLL
- Это когда мы загружаем библиотеку в память Процесса. 
- Для этого нам нужен lib-файл (библиотека импорта)
- Нам нужно знать имена вызываемый функци и какое в данной библиотеке функции библиотеки используется Соглашение о вызовах функций _cdecl, _pascal, _stdcall / Windows (см. выше)

**СУТЬ**: Фактически библиотека с её содержимым приростает к программе, загружаясь полностью в адресное пространство Программы, становится её частью

```
DLL       ПРОГА           DLL            ПРОГА
_______   ______  С   ______________     _______________
|ф-ции | |      | Т  | (фция_библы)-|---|->(фция_библы) |
|библы | |      | => |              |   | адресное      |
|      | |      | А  |              |   | пространоство |
|______| |______| Т  |______________|   |_______________|
```

### Динамический импорт библиотек DLL
- Для обращения к функциям библиотеки "подгружаем" каждую из них когда нам надо через спец. функции:
0) HMODULE GetModuleHandle ( LPCTSTR lpModuleName ) // Имя модуля (если грузим модуль)
1) HINSTANCE LoadLibrary ( LPCTSTR lpLibFileName ) // Имя файла DLL (если грузим DLL)
2) FARPROC GetProcAddress ( HMODULE hModule, LPCSTR lpProcName) // получить адрес функции  // указатель на модуль // имя функции
3) BOOL FreeLibrary( HMODULE hLibModule ); // закончили работу с DLL / модулем

**СУТЬ**: Мы каждый раз через чпец функции обращаемся к библиотеке и вызываем её функции, НО наш процесс не подгружает всю библиотеку в своё адресное пространство, использует лишь те функции что нужно динамически подгружая то, что надо используя функционал библиотеки отдельно от нашей основной программы.

```
                           ___________________________________________
			   |                                         |
                           v                                         |
DLL       ПРОГА           DLL                  ПРОГА                 |
_______   ______  Д   ____________      __________________________   |
|ф-ции | |      | И  | (фция_1)<--|--   | 1.  LoadLibrary()-------|--| 
|библы | |      | => |            | |---|-2.1 GetProcAddr(фция_1) |
|______| |______| Н  | (фция_1)<--|-----|-2.2 GetProcAddr(фция_2) |
                  А  |            |  |--|-2.N GetProcAddr(фция_N) |
		  М  | (фция_N)<--|--|  | 3.  FreeLib()           |
		     |____________|     |_________________________|
```




	
### Ошибки в программе под ОС Windows

Информация об ошибках
0. Логирование в файл событий  
+ рабочее решение
- замедляет работу программы
- нужно вовремя включить
- не всегда есть гарантия записи в файл

Две категории ошибок:
- 1) Падение
- 2) Зависание

- Дедлок / Поток / Процесс завис / зацикливание  
- Если зависло у нас - то на этапе дебага, можем использовать WinDbg -> Attach to process
- Если процесс завис у пользователя можем попробовать сделать то же самое, НО есть его у него нет
- Через диспетчер задач он может сказать какой у него процесс завис (имя),
- Process Explorer может выдать нам выдать про процесс более подробную инфу.
- Пользователь не имеет прав на УСТАНОВКУ, так что без установки стороннего ПО и нельзя через TeamViewer получить доступ на ПК, у него ЕСТЬ права на Запуск.

Девочка / Мальчик из тех поддержки:

------------------------------------------------------------------
111]  ЗАВИСАНИЯ

1) Без подключения и без запуска:

- EventLog виндовый посмотреть
- Создать файл дампа через Диспетчер задач - на рабочем зависшем или независшем короче ещё работающем процессе.
Снять несколько "отпечатков/слепков"файлов дампа процесса. По ним (10 штук) можно примерно понять что происходит.

2) С запуском чего-то от нас:
- ProcessExplorer - Дампы + общая миниинфа
- ProcessMonitor - Полный лог работы этого процесса
    -- куда лезет
    -- какие мьютексы открывает
    -- call стеки какие вызывает
	
Полное логирование в общем + дамп => отсюда можно практически всё понять в чём проблема.

ProcessMonitor - профилировщик удалённый, который аттачится к процессу.

МИНУС: Нужны права на установку этого софта.

------------------------------------------------------------------
222]  ПАДЕНИЯ

222.1] В режиме ОС:
- PageFault
- AccesVialation

http://citforum.ru/programming/delphi/delphi_bugs.shtml

AccesVialation
- имя библиотеки / имя dll - где оно произошло ПО такому-то Адресу:
0х123... Как найти процедуру функции, в которой произошло это падение - 
```
Access violation at address <HEX_value>
in module <Application.Exe>.
Read of address <HEX_value_2>
```
- 1. Статический анализ ситуации - чтобы понять ГДЕ - ТОЧКА
- 2. А по ТОЧКЕ делать какие-то выводы.

Если второй адрес при выдачи ошибки - FFFFFFF (или 0000000) Вы можете почти утверждать, что было обращение к объекту, который еще не был создан. Например, вызов метода формы, которая не была создана.
```
procedure TfrMain.OnCreate(Sender: TObject);
var BadForm: TBadForm;
begin
	BadForm.Refresh; // причина  ошибки
end;
```
	
DLL: Во время сборки проекта так же 
- lib
- idl
- map

- PDB - Programm Data Base - инфа о скомпилиоованной программе.
    -- вызовы функций
    -- параметры
    -- смещеня 

В случае анализа ДАМПА + PDB файл становится очень информативен.
- map - Файл текстовый со всемми точками входа для всех функций, сгенерированных в рамках данной библиотеки
- У каждой точки входа есть своё смещение 

ИТАК Получив Адрес из Исключения - Мы открывваем МЭП файл, там написано с какого смещения начинается какая функция
мы берём этот адрес смотим в какой диапазон / функцию он попадает  => получаем имя функуции и смещение относительно этой функции
открываем дизасемблер смотрим по кодам и получаем кокнретную точку, где и что происзошло, с какими ПАРАМЕТРАМИ.

Это СТАТИЧЕСКИЙ анализ - можем попробовать повторить ситуацию, повызывать функцию именно с такими параметрами и попытка воспросизвести ошибку.


222.2] Внутренние ошибки:
- Exeptions
- Некорректное поведение (внутренняя логика ПО)

Кокнретный шаг - Exception 
- имя библиотеки / имя dll - где оно произошло 
- Адрес, который нам это исключение выдало 



## УТЕЧКИ ПАМЯТИ - MEMORY LEAKS DETECTION - РАЗРАСТАНИЕ ПАМЯТИ ПРОЦЕССА ВО ВРЕМЯ РАБОТЫ

https://habrahabr.ru/post/82514/

CRT

https://rsdn.org/article/vcpp/leaks.xml

Активизация режима обнаружения утечек памяти

Самые главные ваши помощники в борьбе с утечками памяти - отладчик и функции отладочной "кучи" из CRT. Чтобы активизировать эти функции, нужно включить в программу следующие строки:

```
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
```
После того, как вы добавили в вашу программу указанные строчки, вы можете получить информацию об утечках памяти, обратившись к функции
```
_CrtDumpMemoryLeaks();
```
Когда программа выполняется под управлением отладчика, _CrtDumpMemoryLeaks отображает информацию об утечках памяти на вкладке Debug окна Output. Эта информация выглядит примерно так:
```
Detected memory leaks!
Dumping objects ->
C:\PROGRAM FILES\VISUAL STUDIO\MyProjects\leaktest\leaktest.cpp(20) : {18} normal block at 0x00780E80, 64 bytes long.
 Data: <                > CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD 
Object dump complete.
```
Если бы вы не включили в программу директиву #define _CRTDBG_MAP_ALLOC, отчёт выглядел бы так:
```
Detected memory leaks!
Dumping objects ->
{18} normal block at 0x00780E80, 64 bytes long.
 Data: <                > CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD 
Object dump complete.
```
Как видите, _CrtDumpMemoryLeaks выдаёт гораздо более полезную информацию, когда символ _CRTDBG_MAP_ALLOC определён. Без него вам выдаются следующие данные:

- порядковый номер распределения памяти (в фигурных скобках).
- тип блока (нормальный, клиентский или CRT-блок).
- адрес блока (в шестнадцатеричном представлении).
- размер блока в байтах.
- содержимое первых 16 байт этого блока (также в шестнадцатеричном представлении).

Если _CRTDBG_MAP_ALLOC определён, вам дополнительно показывается имя файла, в котором произошло распределение памяти. После имени файла в скобках содержится номер строки (20 в нашем примере). Если сделать двойной щелчок на строчке, содержащей имя файла и номер строки в нём:
```
C:\PROGRAM FILES\VISUAL STUDIO\MyProjects\leaktest\leaktest.cpp(20) : {18} normal block at 0x00780E80, 64 bytes long.
```	
то курсор переместится на строку в файле с исходным кодом программы (в нашем примере строку 20 в файле leaktest.cpp), где произошло распределение памяти. Аналогичного эффекта можно добиться, выделив строчку и нажав F4.

Использование _CrtSetDbgFlag

Если ваша программа всегда завершается в одном и том же месте, использовать _CrtDumpMemoryLeaks достаточно просто. А что делать, если программа может завершаться в различных местах? Вместо того, чтобы расставлять по всей программе вызовы _CrtDumpMemoryLeaks, вы можете разместить в начале программы следующий код:

	_CrtSetDbgFlag( _CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);
	
После этого функция _CrtDumpMemoryLeaks будет автоматически вызываться перед завершением работы вашей программы. Вы должны задать оба флага (_CRTDBG_ALLOC_MEM_DF и _CRTDBG_LEAK_CHECK_DF), как это показано выше.

- Установка режима сообщений CRT
- Установка точки останова на нужном распределении памяти
- Сравнение состояний памяти

Чтобы определить, не было ли утечек памяти на каком-то участке вашей программы, вы можете сделать "снимки" памяти до и после этого участка, а затем вызвать _CrtMemDifference, чтобы сравнить два состояния:
```
_CrtMemCheckpoint( &s1 );
// memory allocations take place here
_CrtMemCheckpoint( &s2 );

if ( _CrtMemDifference( &s3, &s1, &s2) ) 
   _CrtMemDumpStatistics( &s3 );
```

	
	
	
===================================================================

	
# TASKS / ЗАДАЧКИ

1. Развернуть число
2. Развернуть строку
3. Возведение в степень
4. Фактореал
5. перевод строки в число = atio "своими руками"
6. перевод числа в строку
7. Из строки с пробелами удалить все пробелы в строке
8. Найти подстроку в строке
9. Алгоритм Бинарного поиска / Бинарный/Двоичный поиск С++
10. Поиск пузырьком


### 1. Развернуть число

```
int reverseStr(int& n) //tCmplx: O(N) //Space: O(1)
{
  int rev = 0, remainder = 0;
  while(n != 0) {
    remainder = n % 10;
    rev = rev * 10 + remainder;
    n /= 10;
  }
  return rev;
}
```

### 2. Развернуть строку

```
void reverseStr(string& str) //tCmplx: O(N) //Space: O(1)
{
    int n = str.length(); // Swap character starting from two corners
    for (int i = 0; i < n / 2; i++) {
        std::swap(str[i], str[(n - 1) - i]);
		//or
		//char temp = str[i];
		//str[i] = str[ (len-1) - i];
		//str[(len-1) - i] = temp;
	}
}
//#include <bits/stdc++.h> and std::reverse(str.begin(), str.end()); // Reverse str[begin..end]
// OR  Use of reverse iterators => string rev = string(str.Rbegin(), str.Rend());
```

### 3.1 Возведение в степень - Возведение в степень (Итеративно) (Доп.)
A: IN 2^4 OUT 16 (2x2x2x2)

```
int my_pow(int x, unsigned int power) // num^pow
{
  if (power == 0) return 1;
  if (power == 1) return x;
  int n = 15;
  while ((power <<= 1) >= 0) n--;
  long tmp = x;
  while (--n > 0)
    tmp = tmp * tmp * (((power <<= 1) < 0) ? x : 1);
  return tmp;
}
```

### 3.2 Возведение в степень - Возведение в степень (Итеративно) (без учёта отрицательных тепеней)
A: IN 2^4 OUT 16 (2x2x2x2)

```
int my_pow(int n, unsigned int p) // num^pow
{
  int c = 1;
  for(int i=0; i<p; i++)
    c = n * c;
	
  return c;
}
```

### 3.3 Возведение в степень - Возведение в степень (Итеративно) (с учётом отрицательных степеней)
A: IN 2^4 OUT 16 (2x2x2x2)

A: IN 2^-2 OUT 0 (1/4 = 1/(2x2))
```
int my_pow(int n, int p) // num^pow
{
  if(p<0) // 2^(-2) = 1/(2^2)
    p=(-1)*p;
   
  int c = 1;
  for(int i=0; i<p; i++)
  {
    c = n * c;
  }

  if(p<0) // 2^(-2) = 1/(2^2)
     c = 1 / c;

  return c;
}
```

### 3.4 Возведение в степень - Возведение в степень (Улучшенный по скорости)
A: IN 2^6 OUT 64 (2x2x2x2x2x2)

A: IN 2^5 OUT 32 (2x2x2x2x2)
```
int my_pow(int n, int p) // num^pow
{
  if(p<0) // 2^(-2) = 1/(2^2)
    p=(-1)*p;

  int c = 1;
  if(p % 2 == 0) // степень чётная => возводим до половины и полвинное умножаем на само себя
  {
    for(int i=0; i<p/2; i++)
    {
      c = n * c;
    }
    c = c * c;
  }
  else // степень не чётная
  {
    for(int i=0; i<p; i++)
    {
      c = n * c;
    }
  }

  if(p<0) // 2^(-2) = 1/(2^2)
     c = 1 / c;

  return c;
}
```

### 4.1 Факториал (Итеративный) A: IN 5 OUT 125 (1x2x3x4x5)
```
int my_fact(int b)
{
  if(b<0) return 0; // или ОШИБКА !
  
  int c = 1;
  for(int i=1; i<b+1; i++)
  { c = c * i; }
  return c;
}
// tComplx: O(b) | Space O(1)
// (-) works slower then the recursive solution
// (+-) 2 edges of a stick: + more code easier to debug, - more code then in a recursion and requires extra container and or space
// (+) no stackoverflow problem, and Space complexity is O(1)
```
### 4.2 Факториал (Рекурсивный) A: IN 5 OUT 125 (1x2x3x4x5)
```
int my_fact(int num)  // вычисление факториала числа num
{
  if (num <= 1)
	return 1;  // если число не больше 1, возвращаем 1
	
  return (num * fact(num - 1));  // рекурсивный вызов для числа на 1 меньше
}
// tComplx: O(b) | Space O(b)
// (+) works faster, since we work with the stack memory
// (+-) 2 edges of a stick: + less code / compact solution, - too compact, not perfect for debugging
// (-) stackoverflow problem - since recurs leaves on a stack a return addres (to do the return call) to a function, on each step of the recursion level
```
	
### 4.3 Фактореал - Факториал на Шаблонах

```
template<unsigned N> struct factorial {
 static const unsigned value = N * factorial<N - 1>::value;
};
template<> struct factorial<0> {
 static const unsigned value = 1;
};

// Пример обращения:
const unsigned f5 = factorial<5>::value; // 120
```

### 5.1 перевод строки в число =  (1) atio "своими руками"
A: IN строка str="345" OUT в число типа int n=435

```
int myAtio(std::string a = "456")
{
  int c = 0;
  int len = a.size();
  
  for(int i=0; i < len; i++)
  {
    c = c*10;
	switch(a[i])
	{
	  case '0': c=c+0; break;
	  case '1': c=c+1; break;
	  case '2': c=c+2; break;
	  case '3': c=c+3; break;
	  case '4': c=c+4; break;
	  case '5': c=c+5; break;
	  case '6': c=c+6; break;
	  case '7': c=c+7; break;
	  case '8': c=c+8; break;
	  case '9': c=c+9; break;
	}
  }
  return c;
}
```
		       
### 5.2 перевод строки в число = atio "своими руками"
A: IN строка str="345" OUT в число типа int n=435

```
int myAtio(std::string a = "456")
{
  int c = 0;
  int len = a.size();
  for(int i=0; i < len; i++)
  {
    c = c * 10;
	c = (a[i] - '0') + c;
  }
  return c;
}
```

### 6. перевод числа в строку
```
1. Time Complexity: O(n) Auxiliary Space: O(n)
 int num = 2016;
 ostringstream str1;
 str1 << num;
 string geek = str1.str();

2. Time Complexity: O(n) Auxiliary Space: O(n)
 int i_val = 20;
 string stri = to_string(i_val);
  
3.
 string strf = boost::lexical_cast<string>(f_val);
```

### 7. Из строки с пробелами удалить все пробелы в строке
```
class DeleteAllSpacesFromString
{
public:
    DeleteAllSpacesFromString();
    ~DeleteAllSpacesFromString();
    
    /// Решение с выделением доп памяти
    std::string deleteAllSpacesFromString(const std::string& s)
    {
        std::string res;
        for (auto symbol: s) {
            if(symbol != ' ') {
                res.push_back(symbol);
            }
        }
        return res;
    }

    /// Решение БЕЗ выделения доп памяти
    void deleteAllSpacesFromStringExtra(std::string& s)
    {
        for (int i = 0; i< s.size(); ++i) {
            if (s[i] == ' ') {
                for (int j = i + 1; j < s.size(); ++j) {
                    if (s[j] != ' ') {
                        s[i] = s[j];
                        s[j] = ' ';
                    }
                }
            }
        }
    }

};
	
```

### 8. Найти подстроку в строке

```
#include <string>
#include <iostream>

using namespace std;

template<typename T>
void removeSubstrs
(
  basic_string<T>& str,
  const basic_string<T>& psub
)
{
  basic_string<T>::size_type n = psub.length();

  for( basic_string<T>::size_type i = str.find(psub);
       i != basic_string<T>::npos;
       i = str.find(psub)
  )
  {
    str.erase(i, n);
  }
}

int main()
{
   string s = "One fish, two fish, red fish, blue fish";
   string p = "fish";
   removeSubstrs(s, p);
   cout << s << '\n';
}
	
```

### ПРОВЕРИТЬ
```
int binary_find(int n, int *x, long A)
{
 int m, left, right;
 left = 0; right = n-1;
 while (true)
 {
    if (left > right) return (-1); // значение не найдено
    m = left + (right - left) / 2;
    if (x[m] < A) left = m + 1;
    if (x[m] > A) right = m - 1;
    if (x[m] == A) return m;
 }
}
```

### 9. Алгоритм Бинарного поиска / Бинарный/Двоичный поиск С++
### 9. Recursive implementation of Binary Search tCmplx: O(log n) ; Space: O(1)
```
int binarySearch(int arr[], int l, int r, int x)
{
  if (r >= l) {
      int mid = l + (r - l) / 2;
      // If the element is present at the middle itself
      if (arr[mid] == x)
          return mid;
      // If element is smaller than mid, then
      // it can only be present in left subarray
      if (arr[mid] > x)
          return binarySearch(arr, l, mid - 1, x);
      // Else the element can only be present
      // in right subarray
      return binarySearch(arr, mid + 1, r, x);
  }
  // We reach here when element is not present in array
  return -1;
}
 
int main(void)
{
    int arr[] = { 2, 3, 4, 10, 40 };
    int x = 10;
    int n = sizeof(arr) / sizeof(arr[0]);
    int result = binarySearch(arr, 0, n - 1, x);
}
```

### 9. Алгоритм Бинарного поиска / Бинарный/Двоичный поиск С++
### 9. Iterative implementation to Binary Search tCmplx: O(log n) ; Space: O(1)
```
int binarySearch(vector<int> v, int To_Find)
{
  int lo = 0, hi = v.size() - 1;
  int mid;
  // This below check covers all cases , so need to check
  // for mid=lo-(hi-lo)/2
  
  while (hi - lo > 1) { // while(lo<=hi)
      int mid = (hi + lo) / 2;
  	
      if (v[mid] < To_Find) {
          lo = mid + 1;
      }
      else {
          hi = mid;
      }
  }
  
  if (v[lo] == To_Find) { cout << "Found" << " At Index " << lo << endl; }
  else if (v[hi] == To_Find) { cout << "Found" << " At Index " << hi << endl; }
  else { cout << "Not Found" << endl; }
}
 
int main()
{
  vector<int> v = { 1, 3, 4, 5, 6 };
  int To_Find = 1;
  binarySearch(v, To_Find);
}
```
	
### 10.1 Сортировка  пузырьком
```
template<typename T>
void bubble_sort(T array[], std::size_t size)
{
  for (std::size_t idx_i = 0; idx_i < size - 1; idx_i++)
  {
    for (std::size_t idx_j = 0; idx_j < size - idx_i - 1; idx_j++)
    {
      if (array[idx_j + 1] < array[idx_j])
      {
        std::swap(array[idx_j], array[idx_j + 1]);
      }
    }
  }
}
```
### 10.2 Сортировка  пузырьком улучшенный
```
void bubbleSort(int* arrayPtr, int arrLen) // сортировка пузырьком
{
 int temp = 0; // временная переменная для хранения элемента массива
 bool exit = false; // булевая переменная для выхода из цикла, если массив отсортирован
 
 while (!exit) // пока массив не отсортирован
 {
  exit = true;
  for (int i = 0; i < (arrLen - 1); i++) // внутренний цикл
    //сортировка пузырьком по возрастанию - знак >
    //сортировка пузырьком по убыванию - знак <
    if (arrayPtr[i] > arrayPtr[i + 1]) // сравниваем два соседних элемента
    {
     // выполняем перестановку элементов массива
     temp = arrayPtr[i];
     arrayPtr[i] = arrayPtr[i + 1];
     arrayPtr[i + 1] = temp;
     exit = false; // на очередной итерации была произведена перестановка элементов
    }
 }
}
```

### 11. Развернуть linked_list
``` 
// Link list node
struct Node {
    int data;
    struct Node* next;
    Node(int data)
    {
        this->data = data;
        next = NULL;
    }
};
 
struct LinkedList {
    Node* head;
    LinkedList() { head = NULL; } 
	
    // Function to reverse the linked list
    void reverse()
    {
        // Initialize current, previous and next pointers
        Node* current = head;
        Node *prev = NULL, *next = NULL;
        while (current != NULL) {
            next = current->next; // Store next
            current->next = prev; // Reverse current node's pointer
            prev = current; // Move pointers one position ahead.
            current = next;
        }
        head = prev;
    }
 
    void push(int data)
    {
        Node* temp = new Node(data);
        temp->next = head;
        head = temp;
    }
};
 
/* Driver code*/
int main()
{
    /* Start with the empty list */
    LinkedList ll;
    ll.push(20);
}
```

===================================================================
	
	
# PATTERN PATTERNS ШАБЛОНЫ ПРОЕКТИРОВАНИЯ СИСТЕМНЫЙ ДИЗАЙН ПАТТЕРНЫ

https://en.wikipedia.org/wiki/Double-checked_locking

С С++11 проблемы больше для поддержки потоков ничего и не надо. Но до полной его поддержки всеми компиляторами надо ещё дожить.
```	
Singleton& GetInstance() {
  static Singleton s;
  return s;
}
```
	
### Q: Синглтон Класс (небезопасный в многопоточном окружении до С++11)
```
// Singleton.h
class Singleton
{
  private:
    static Singleton * p_instance;
    // Конструкторы и оператор присваивания недоступны клиентам
    Singleton() { }
    Singleton( const Singleton& );  
    Singleton& operator=( Singleton& );
  public:
    static Singleton * getInstance() {
        if(!p_instance)           
            p_instance = new Singleton();
        return p_instance;
    }
};
```

### Q: Синглтон Мейерса / Майерса (небезопасный в многопоточном окружении  до С++11) КОД 
```
// Singleton.h
class Singleton
{
private: 
    Singleton() {}
    Singleton( const Singleton&);  
    Singleton& operator=( Singleton& );
public:
    static Singleton& getInstance() {
        static Singleton  instance;
        return instance;
    }    
}; 
```	

### Q: Синглтон Дейкстры (использующий метода "Double-Checked Locking" для threadsafity) 
Ранее проблему решали с использованием метода "Double-Checked Locking" / "Double-Checked Locking Pattern" / Блокировка с двойной проверкой, хоть он и был более затратен.
По причине возникновения такого решения, синглтон с использованием "Double-Checked Locking" назвали Синглтон Дейкстры.	

```
#include <atomic>
#include <mutex>

class Singleton {
 public:
  static Singleton* GetInstance() 
  {
    Singleton* p = s_instance.load(std::memory_order_acquire);
    if (p == nullptr) { // 1st check
      std::lock_guard<std::mutex> lock(s_mutex);
      p = s_instance.load(std::memory_order_relaxed);
      if (p == nullptr) { // 2nd (double) check
        p = new Singleton();
        s_instance.store(p, std::memory_order_release);
      }
    }
    return p;
  }	

 private:
  Singleton() = default;

  static std::atomic<Singleton*> s_instance;
  static std::mutex s_mutex;
};

```	

**CONS in general: OVERENGENEERING**

- **Abstract Factory** - Lets you produce families of related objects without specifying their concrete classes.
- **Builder** - Lets you construct complex objects step by step. The pattern allows you to produce different types and representations of an object using the same construction code.
- **Factory Method** - Provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created.

- **Singleton** – having only 1 instance of an object. – it’s OK if it’s not shared it’s perfect to get access from allover the place to this entity. A Singleton encapsulates a unique resource and makes it readily available throughout the application.	
- **Singleton** – shared resource [stored data] multiple objects must have access to that data, here we start to have race conditions (here we can use mutexes, cond-vars, double check pattern approach)	
- **Singleton** – extra problem – it can become a SUPERCLASS, that knows everything about everyone and every software part and software layer can have access to any variable or object that’s inside the Singleton (Laser-class).

- **Singleton** –  We can have any type of problems – starting from INCONSISTANCY OF VARIABLE STATES we read a buffered value, that has already changed it’s state on the device, finishing with – if it’s an shared resource and accessed from MULTIPLE THREADS at the same time, and it has a synchronization object we can get a deadlock
- **Facade** - Provides a simplified interface to a library, a framework, or any other complex set of classes.

Cons: Façade is usefull, when that simplified interface at the end really simplifies your life. We had a teammate who created 100 façade classes, each of them had a derived class, and our college sad when you will need the functionality of each of them, think in advance – Because 1) if you will be able to use my interface and subclass from it, then do it in the first place. 2) if you extremely need to subclass from the original class and you do not want to touch int interface, it’s up to you, but in this case think, maybe you are doing something wrong…
Strong CONNECTEDNESS, 

- **Template Method** - Defines the skeleton of an algorithm in the superclass but lets subclasses override specific steps of the algorithm without changing its structure.

- **State** - Lets an object alter its behavior when its internal state changes. It appears as if the object changed its class.,
- **Visitor** - Lets you separate algorithms from the objects on which they operate.
- **Strategy** - Lets you define a family of algorithms, put each of them into a separate class, and make their objects interchangeable.

- **Oberver or Listener** bei Client Server Applicationen
The Observer looks for sertain amount of Actions and has appropriate ActionHandlers classes, that are called when the action accures. 
```	
_______________________->update() ------------------------------------ ObservA()
[Subject->Subjects changes]->update()] ObserverBase with notify() -> ObservB()
_______________________->update() --------------------------------------> ObservC()
```
- **Command** - Turns a request into a stand-alone object that contains all information about the request. This transformation lets you pass requests as a method arguments, delay or queue a request's execution, and support undoable operations.
- **Iterator** - Lets you traverse elements of a collection without exposing its underlying representation (list, stack, tree, etc.).
- **Proxy** - Lets you provide a substitute or placeholder for another object. A proxy contr- ols access to the original object, allowing you to perform something either before or after the request gets through to the original object.- **

- **Adapter** - Allows objects with incompatible interfaces to collaborate.
- **Composite** - Lets you compose objects into tree structures and then work with these structures as if they were individual objects.
- **Prototype** - Lets you copy existing objects without making your code dependent on their classes.
- **Decorator** - Lets you attach new behaviors to objects by placing these objects inside special wrapper objects that contain the behaviors.
	
	
===================================================================
	
	
# HISTORY	
	
Was das fachliche anbetrifft.

In der IT brounche habe ich als 
- Windows SystemAdministartor zu arbeiten. Meine Arbeit bestand darin dass ich Oper.Systeme auf die PCs installiert habe, Drucker und andere Geräte angegschlossen, Einrichtung lokaler Netze, etwas ingenior Arbeit. Ich habe mich hier mit BorlandC++ Builder begegnet. War kein richtiges programieren, aber die erste Begegnung mit c++ Sprache und mit IDE.

- Dann im Dritten StudienJahr hatten wir ein Betriebspraktikum bei Kaspersky. Dort leitete man uns in das Project ein, man hat unz gezeigt wie VierenAnalytiker arbeiten, wie Sie die meiste Zeit das untershiedlichste code sehen, VisualBasic code, C++, JavaScript und andere, wie man den Reverce Enegeniering des Codes macht. 

Ich habe rheines C code für einige Teile des Analyseszstems geschrieben, Suchalgorythmen analisiert und sehr viel debugging von code gemacht um Patterns zu entdecken. 

Ich konnte nich angestellt werden, da sie nur nach Seniour SWE suchten, nur Seniour Positionen waren in der Firma offen, und ich hatte noch nicht genug ERFAHRUNG.
SW Enwicklung hat mir noch ab der Uni gefallen. Dort war beides interssant – sowohl das obeflachlicke auf der Client seite alsauch die backend seite der programmierung. Der fakt das man mit code etwas erschaffen kann und das es funkzioniert und alles was man dafür braucht ist – nur code zu schreiben. Obwohl wir in dem 1-sten StudienJar nur rheines C, Pascal und Visual Basic hatten, trotzdem, als man mir gezeigt hat wie man diese Basis Kentnisse in der Realität nutzt und das man damit große software produkte ertellen kann, ich wollte nur mehr und tiefer mich damit beschäftigen und weiter als SWE zu arbeiten.

- Als SW Enwitckler habe in der Firma Symbol Designe (Task in Task out, Kanban)
Windows Client Server Application – Es waren mehrere SCADA-Systeme für Fixierung von Erdölleck auf der Erdölleitung.  Die mit C++ entwickelt worden waren. Wieso – weil man jeden Protokol unterschtützen konnte, und Maximale flexibilität.  Viel GUI-Programmierung (MFC classen, BCGP bibl) und etwas Multi-Threading mit standarten STL-Funktionalität und deren Objekte std::condition_variable, std::thread, deque, algorithm Transport der Daten zwischen dem Server und den Geräten (Cleine SCADA Densitometer, Controller, Druckmäsgeraten) war OPC und COM/DCOM um mit C# server von ein par Firmen extra zu kommunizieren.  Die Firma war zu klein, nach drei Jahren habe ich gespürt, dass ich nicht wachse, und ich wollte mich weiter entwickeln und mehr Verdienen

- C++ Softwareentwickler Encore GmbH, Moskau. (Kleines Team aus 2 Entwickler erste begegnung mit Scrum)
- Ich habe mich der Entwicklung eines Benutzer Authentifizierungssystem für Windows und Astralinux (Credential Provider, dadurch erlaubt Windows den Winlogon zu customisieren) angeschlossen. Ich habe extra den Teil für das einloggen mit CryptoToken geschrieben. Viel WINAPI + CryptoAPI + Nur BackendSeite. Dazu noch extra Entwicklung von Bibliotheken und Windows-Services die private Benutzer-Informationen spechern und bearbeiten. (Token Zugriff und etc.)
- Entwicklung von Integritätskontrollsystem von Dateien (Ordnern) und Windows-Registrierungsdatenbank. In diesem Projekt habe ich persönlich - GUI auf QT entwickelt und den Teil für Integritätskontrolle von Registrierungsdatenbank auf mich genommen

Ich habe kurz auf dieser Stelle gerabeitet, weil die Firma sich geschlossen hat. Es hatte mehr mit der Arbeit der Manager und einer nicht richtigen Posizionierung der Firma auf dem Markt, und die Firma ist Pleite gegangen. Und ich musste meine Arbeit wechseln.

- C++ Leading Developer RGS GROUP OAG, Moskau – zu einfache Arbeit, zu alte Technologien, und kein Wachstum als Spezialist ... CRM - customer relationship management - Nicht das ich von jeder Arbeit weggeehe wo es mir zu einfach alles scheint. Ich habe 2 bis 3 Wochen nicht gearbeitet und keine Aufgaben gebat. Der Grund dafür war der Waterfall-Prinziep. Es was ein altes Produrk, einige APIS waren noch in 1995 geschrieben. Es war eiene Große Firma, und sie stellte BtoB produkte, also Ihre Kunden waren kleinere Versicherungs-Firmen. Meistens machte der Kunde einfach 3 – 10 screenshots von unterschiedlichster Software, hat die Technische Aufgabe beschrieben und, danach haben die Analytiker von dem Kunden extra Information über das Produkt gesammelt. Am nächsten Schritt machten vor ein Entwurf der DatenBank (zusammen mit den DatenBank Spetialisten), oder wir habe besprochen wie wir das in das schon existierte szstem integrieren konnten. SoftwareDesign - GUI-Makett wurde entworfen und die Business logig beschrieben. Ich bekam (WATERFALL tief durchVerabeitete TA) eine 200 seitige Technische Aufgabe, und man musste es innerhalb von 1 oder 2 Monate alles fertig stellen.
	
ERP/CRM/CAD/CAM-System vorbestimmt für Berechnungen von Versicherungslebensmitteln. Entwicklung von Backend in C++ und GUI mit MFC-Framework in MS Visual Studio 2013 (neue Apps mit Qt (QML) in QtCreator)
Ein bischen habe ich dort auch mit Neuronetzen gearbeitet, aber diese Abteilung der Firma hat sich zu spät organisiert und ich hatte zu diesem Zeitpunkt schon einen JobOffer von der TochterFirma der Russischen Bahn auf der Hand

Es waren etwas zu kurze Stationen,  der Grund dafür lag daran das ich 
- 01/2018 – bis heute C++/Qt Leading Software Engineer Locotech-Signal Moskau	TochtorFirma der Russischen Bahn
Client- und Serverseiten von SCADA für Eisenbahnverkehrsysteme.

1 Jahr habe ich als normaler SoftwareEngenier gearbeitet und dann Habe ich ein TeamLead 4 Entwicklern geleitet. Ich war ScrumMaster, habe scrum Meetings geführt. Das übliche halt, Iteration gestartet, Große Aufgabe besprochen, sie in kleinere Aufgaben aufgeteilt - Dekomposition gemacht, nachdem ware Aufgaben auch dem Scrum-Board, Arbeits-zeiten für jede aufgabe geplant und fixiert and Iteration startete. Hier arbeiteten wir nach einem normalen Scrum, mit allen Teilnehmer des Entwicklungsprozess: Analytiker, UI/UI-Designer, SoftwareEntwickler, Tester

1. ClientServerApp 2. StationDesigner – CAD-System 3. Logs Player 

Eisenbahnverkehrsysteme. GUI Implementierung mit Qt Widgets, tiefe benutzerdefinierte Implementierung. Entwicklung von Visualisation des Eisenbahnverkehrs, als ein QtQuickView-Komponent mit standarten und selbst entwickelten benutzerdefinierten QtQuick-Controls (viel QML). 80% von allen Projekten der Firma sind mit Qt-Framework geschrieben (nicht nur Qt GUI-Classen sondern auch Qt-Container / Threading Classen / QSqlDatabase / QModbus und andere.

Client - QMLScene in der Mitte mit Widgets umgeben - Treeviews mit statinen links, Alarame Warnungen Informations Messeges unten in ter TabellenViw, Rects die wichtingsten Knöpfe,

- Senior Software Engineer at "IPG Photonics",
Reimplemented old Singleton-based application solution by adding more suitable application and communication layers. 
Singleton – extra problem – it can become a SUPERCLASS, that knows everything about everyone and every software part and software layer can have access to any variable or object that’s inside the Singleton (Laser-class).

- Singleton –  We can have any type of problems – starting from INCONSISTANCY OF VARIABLE STATES we read a buffered value, that has already changed it’s state on the device, finishing with – if it’s an shared resource and accessed from MULTIPLE THREADS at the same time, and it has a synchronization object we can get a deadlock
Singleton – having only 1 instance of an object. – it’s OK if it’s not shared it’s perfect to get access from allover the place to this entity. A Singleton encapsulates a unique resource and makes it readily available throughout the application.
- Singleton – shared resource [stored data] multiple objects must have access to that data, here we start to have race conditions (here we can use mutexes, cond-vars, double check pattern approach)
- SOLUTION WAS – we just have sent the commands not in Asyncronious manner but in Syncronious manner. There for we could right away have time to wait for the answer, if we did not got it => means the device, board, firmware, or something else was broken either way and we had to shut down the system. The syncronious manar also gave us the possibility easy to send multiple commands always in a correct and for sure known order. And we have sent the commands not from the Superclass that contained each and every state, but we just called it inside of thar particular place, where these commands were really needed. If we had to know anything else abot multiple systemparts or their states, we always could pass a pointer of that instance, we could collect multiple pointers in a deccisionManagerClass,  use a proxy class ETZETERA it was up to the situation

Developed a line following algorithm for a laser knife which uses CAN Bus to communicate with camera and the laser.
The ALGO had 3 steps:
1) First there was a laser pointer, and it pointed out the way of the laser knife. We pressed the button and the system SAVED the “way” for the knife. Then we have chosen Frequency, Power and Pulse Energy for the laser, and how deep should the knife go in. 
2) Then The one thread got the information about the knife positions and compared it to the positions of the way, that the knife should go, following the vector that was calculated as the nearest waypoint between 2 points
WE had only a CONDITION_VARIABLE to notify both threads that something went wrong.
3) The laser pointer in the other thread was watching for the organ, and was watching if there will be any unexpected movement during the procedure.
4) These 2 threads did not have any shared objects or shared memory, so there was not that much of concurrency stuff to be handled.

CONS in general: OVERENGENEERING


