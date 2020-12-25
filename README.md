#ifndef ANSWER_H
#define ANSWER_H

#include <QString>

class Answer {
private:
    QString answer;
    int result;
public:
    explicit Answer(QString answer = "", int result = 0) : answer(answer), result(result){}

    QString getAnswer() { return answer; }
    void setAnswer(QString answer) { this->answer = answer; }

    int getResult() { return result; }
    void setResult(int result) { this->result = result; }
};

#endif // ANSWER_H

#ifndef FILE_H
#define FILE_H

#include "user.h"
#include "question.h"
#include "answer.h"
#include "List.h"
#include <iostream>
#include <fstream>
#include <string>

using namespace std;

class File {
public:
    static void openQuestion(Question *question, string file) {
        ifstream in(file);
        while (!in.eof()) {
            string quest;
            getline(in, quest);
            if (quest == "") break;
            question = new Question(QString::fromStdString(quest));
        }
        in.close();
    }

    static void saveQuestion(Question *question, string file) {
        ofstream on(file);
        on << question->getQuestion().toStdString();
        on.close();
    }

    static void openAnswer(List<Answer> *answerList, string file) {
        ifstream in(file);
        while (!in.eof()) {
            string answer;
            int result;
            getline(in, answer);
            if (answer == "") break;
            in >> result;
            in.get();
            Answer newAnswer(QString::fromStdString(answer), result);
            answerList->pushTail(newAnswer);
        }
        in.close();
    }

    static void saveAnswer(List<Answer> *answerList, string file) {
        ofstream on(file);
        for (int i = 0; i < answerList->size(); i++) {
            on << answerList->operator[](i).getAnswer().toStdString() << endl;
            on << answerList->operator[](i).getResult() << endl;
        }
        on.close();
    }

    static void openUser(List<User> *userList, string file) {
        ifstream in(file);
        while (!in.eof()) {
            string username;
            bool active;
            getline(in, username);
            if (username == "") break;
            in >> active;
            in.get();
            User newUser(QString::fromStdString(username), active);
            userList->pushTail(newUser);
        }
        in.close();
    }

    static void saveUser(List<User> *userList, string file) {
        ofstream on(file);
        for (int i = 0; i < userList->size(); i++) {
            on << userList->operator[](i).getUsername().toStdString() << endl;
            on << userList->operator[](i).getActive() << endl;
        }
        on.close();
    }
};

#endif // FILE_H

#pragma once
#include <iostream>
#include <iomanip>
#include <QString>
template <class T>
struct Node {
    T data;
    Node<T>* previous;
    Node<T>* next;
    int number;
};

template <class T>
class List {
protected:
    Node<T> *head;
    Node<T> *tail;
    int amount;
public:
    List() { head = nullptr; tail = nullptr; amount = 0; }
    ~List();

    //Длина списка
    int size() { return this->amount; }
    void pushHead(T obj);
    void pushTail(T input_object);
    T popHead();
    T popTail();
    T& operator[](int num);
};

//Добавление с головы
template<class T>
void List<T>::pushHead(T obj) {
    if (head == nullptr) {
        head = new Node<T>;
        head->data = obj;
        head->next = nullptr;
        head->previous = nullptr;
        head->number = (amount++);
        tail = head;
    } else {
        auto *node = new Node<T>;
        node->data = obj;
        node->next = head;
        node->previous = nullptr;
        head->previous = node;
        head = node;
        for (int i = 0;node;node = node->next, i++) {
            node->number = i;
        }
        amount++;
    }
}

//Добавление в хвост
template<class T>
void List<T>::pushTail(T input_object) {
    if (head == nullptr) {
        head = new Node<T>;
        head->data = input_object;
        head->next = nullptr;
        head->previous = nullptr;
        head->number = (amount++);
        tail = head;
        return;
    }
    auto *node = new Node<T>;
    node->data = input_object;
    node->next = nullptr;
    node->previous = tail;
    tail->next = node;
    tail = node;
    node->number = (amount++);
}

//Удаление от головы
template<class T>
T List<T>::popHead() {
    if (!(head)) return T();
    T data = head->data;
    Node<T>* node = head;
    if (head != tail) {
        head = head->next;
        head->previous = nullptr;
        Node<T>* tmp = head;
        for (int i = 0;tmp;tmp = tmp->next, i++) {
            tmp->number = i;
        }
    } else {
        head = tail = nullptr;
    }
    delete node;
    amount--;
    return data;
}

//Удаление из хвоста
template<class T>
T List<T>::popTail() {
    if (!(head)) return T();
    T data = tail->data;
    Node<T> *node = tail;
    if (tail != head) {
        tail = tail->previous;
        tail->next = nullptr;
    } else {
        head = tail = nullptr;
    }
    delete node;
    amount--;
    return data;
}

// Доступ к объектам с помощью индексации
template<class T>
T& List<T>::operator[](int num) {
    Node<T> *curr = head;
    if (num < 0 || num >= amount) return curr->data;
    for (int i = 0; i < num; i++)
        curr = curr->next;
    return curr->data;
}

template<class T>
List<T>::~List() { while (this->head) { this->popHead(); } }


#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QString>
#include "file.h"
#include "menuwindow.h"

QT_BEGIN_NAMESPACE
namespace Ui { class MainWindow; }
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private slots:
    void on_authorization_clicked();

    void on_exit_clicked();

private:
    Ui::MainWindow *ui;
};
#endif // MAINWINDOW_H
#include "mainwindow.h"
#include "ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_authorization_clicked()
{
    QString username = ui->lineEdit->text();
    List<User> *userList = new List<User>();
    File::openUser(userList, "user.txt");
    bool flag = false;
    for (int i = 0; i < userList->size(); i++) {
        if (username == userList->operator[](i).getUsername()) {
            flag = true;
        }
    }
    if (!flag) {
        User newUser(username, false);
        userList->pushTail(newUser);
        File::saveUser(userList, "user.txt");
    }
    MenuWindow *w = new MenuWindow(username);
    w->show();
    this->close();
    delete this;
}

void MainWindow::on_exit_clicked()
{
    QApplication::exit(0);
}

#include "mainwindow.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MainWindow *w = new MainWindow;
    w->show();
    return a.exec();
}

#ifndef MENUWINDOW_H
#define MENUWINDOW_H

#include <QMainWindow>
#include "mainwindow.h"
#include "pollwindow.h"

namespace Ui {
class MenuWindow;
}

class MenuWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MenuWindow(QString username = nullptr);
    ~MenuWindow();

private slots:
    void on_pushButton_3_clicked();

    void on_pushButton_2_clicked();

    void on_pushButton_clicked();

private:
    Ui::MenuWindow *ui;
    QString username;
};

#endif // MENUWINDOW_H

#include "menuwindow.h"
#include "ui_menuwindow.h"
#include "resulttableform.h"

MenuWindow::MenuWindow(QString username) :
    username(username),
    ui(new Ui::MenuWindow)
{
    ui->setupUi(this);
    List<User> *userList = new List<User>();
    File::openUser(userList, "user.txt");
    for (int i = 0; i < userList->size(); i++) {
        if (username == userList->operator[](i).getUsername()) {
            if (userList->operator[](i).getActive()) {
                ui->pushButton_3->setHidden(true);
            }
        }
    }
}

MenuWindow::~MenuWindow()
{
    delete ui;
}

void MenuWindow::on_pushButton_3_clicked()
{
    PollWindow *w = new PollWindow(username);
    w->show();
    this->close();
    delete this;
}

void MenuWindow::on_pushButton_2_clicked()
{
    ResultTableForm *w = new ResultTableForm();
    w->show();
}

void MenuWindow::on_pushButton_clicked()
{
    MainWindow *w = new MainWindow;
    w->show();
    this->close();
    delete this;
}

#ifndef POLLWINDOW_H
#define POLLWINDOW_H

#include <QMainWindow>
#include "menuwindow.h"
#include "mainwindow.h"

namespace Ui {
class PollWindow;
}

class PollWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit PollWindow(QString username = nullptr);
    ~PollWindow();

private slots:
    void on_pushButton_clicked();

    void on_pushButton_2_clicked();

private:
    Ui::PollWindow *ui;
    QString username;
};

#endif // POLLWINDOW_H

#include "pollwindow.h"
#include "ui_pollwindow.h"

PollWindow::PollWindow(QString username) :
    username(username),
    ui(new Ui::PollWindow)
{
    ui->setupUi(this);
    Question *question = new Question;
    File::openQuestion(question, "question.txt");
    ui->label_2->setText(question->getQuestion());
    List<Answer> *answerList = new List<Answer>();
    File::openAnswer(answerList, "answer.txt");
    for (int i = 0; i < answerList->size(); i++) {
        ui->listWidget->addItem(QString::number(i + 1) + ". " + answerList->operator[](i).getAnswer());
    }
    ui->spinBox->setMinimum(1);
    ui->spinBox->setMaximum(answerList->size());
}

PollWindow::~PollWindow()
{
    delete ui;
}

void PollWindow::on_pushButton_clicked()
{
    MenuWindow* w = new MenuWindow(username);
    w->show();
    this->close();
    delete this;
}

void PollWindow::on_pushButton_2_clicked()
{
    List<Answer> *answerList = new List<Answer>();
    File::openAnswer(answerList, "answer.txt");
    int number = ui->spinBox->text().toInt() - 1;
    answerList->operator[](number).setResult(answerList->operator[](number).getResult() + 1);
    List<User> *userList = new List<User>();
    File::openUser(userList, "user.txt");
    for (int i = 0; i < userList->size(); i++) {
        if (username == userList->operator[](i).getUsername()) {
            userList->operator[](i).setActive(true);
        }
    }
    File::saveUser(userList, "user.txt");
    File::saveAnswer(answerList, "answer.txt");
    MenuWindow* w = new MenuWindow(username);
    w->show();
    this->close();
    delete this;
}

#ifndef QUESTION_H
#define QUESTION_H

#include <QString>

class Question {
private:
    QString question;
public:
    explicit Question(QString question = "") : question(question) {}

    QString getQuestion() { return question; }
    void setQuestion(QString question) { this->question = question; }
};

#endif // QUESTION_H

#ifndef RESULTTABLEFORM_H
#define RESULTTABLEFORM_H

#include <QWidget>
#include "menuwindow.h"

namespace Ui {
class ResultTableForm;
}

class ResultTableForm : public QWidget
{
    Q_OBJECT

public:
    explicit ResultTableForm(QWidget *parent = nullptr);
    ~ResultTableForm();

private slots:
    void on_pushButton_clicked();

private:
    Ui::ResultTableForm *ui;
};

#endif // RESULTTABLEFORM_H

#include "resulttableform.h"
#include "ui_resulttableform.h"

ResultTableForm::ResultTableForm(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::ResultTableForm)
{
    ui->setupUi(this);
    ui->tableWidget->setColumnCount(2);
    Question *question = new Question;
    File::openQuestion(question, "question.txt");
    ui->label_2->setText(question->getQuestion());
    List<Answer> *answerList = new List<Answer>();
    File::openAnswer(answerList, "answer.txt");
    ui->tableWidget->setHorizontalHeaderItem(0, new QTableWidgetItem("Вариант ответа"));
    ui->tableWidget->setHorizontalHeaderItem(1, new QTableWidgetItem("Количество голосов"));
    for (int i = 0; i < answerList->size(); i++) {
        ui->tableWidget->insertRow(i);
        ui->tableWidget->setItem(i, 0, new QTableWidgetItem(answerList->operator[](i).getAnswer()));
        ui->tableWidget->setItem(i, 1, new QTableWidgetItem(QString::number(answerList->operator[](i).getResult())));
    }
    ui->tableWidget->resizeColumnsToContents();
}

ResultTableForm::~ResultTableForm()
{
    delete ui;
}

void ResultTableForm::on_pushButton_clicked()
{
    this->close();
    delete this;
}

#ifndef USER_H
#define USER_H

#include <QString>

class User {
private:
    QString username;
    bool active;
public:
    explicit User(QString username = "", bool active = false) : username(username), active(active) {}

    QString getUsername() { return username; }
    bool getActive() { return active; }

    void setUsername(QString username) { this->username = username; }
    void setActive(bool active) { this->active = active; }
};

#endif // USER_H
