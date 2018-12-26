import xlrd
import sqlite3
import os
#################    ПЕРВЫЙ ПУНКТ     #######################
#Создаем общую таблицу по анкетам
conn = sqlite3.connect("credit.db")
cursor = conn.cursor()
cursor.execute("""create table anket(id integer,
               birth real,
               children integer,
               family integer,
               income integer,
               issue real,
               gender text,
               mrtstatus text,
               educ text,
               housing text,
               empl text)""")
for f in os.listdir("C:/ankety/"):
    f1 = "C:/ankety/" + f
    data = xlrd.open_workbook(f1)
    sheet = data.sheet_by_index(0)

    feature = [0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    what = ['Identity Number', 'Date of Birth','Children', 'Family', 'Income', 'Issue date', 'Gender', 'Marital Status', 'Education', 'Housing', 'Employed By']
    stroka = sheet.nrows
    stolbec = sheet.ncols

    for i in range(stroka):
        for j in range(stolbec):
            for k in range(11):
                if sheet.cell(i, j).value == what[k]:
                    feature[k] = sheet.cell(i + 1, j).value



    # SQL
    sql = "insert into anket(id, birth, children, family, income, issue, gender, mrtstatus, educ, housing, empl) VALUES (?, ?, ?, ?, ?,?, ?, ?, ?, ?, ?)"
    cursor.execute(sql, (feature[0], feature[1], feature[2], feature[3], feature[4], feature[5], feature[6], feature[7],feature[8], feature[9], feature[10]))

    sql3 = "commit"
    cursor.execute(sql3)
    print(f)

conn.commit()

#Отобразим таблицу с данными по анкетам
conn = sqlite3.connect("credit.db")
cursor = conn.cursor()
sql = "select * from anket"
cursor.execute(sql)
for row in cursor.execute("select rowid, * from anket"):
    print(row)


#Создаем общую таблицу по контрактам
conn = sqlite3.connect("credit.db")
cursor = conn.cursor()
cursor.execute("""create table contract (id integer,
               contrnumb text,
               amount integer,
               type text,
               term integer,
               annuity integer)""")

import os
for f in os.listdir("C:/contracts/"):
    f1 = "C:/contracts/" + f
    data = xlrd.open_workbook(f1)
    sheet = data.sheet_by_index(0)

    feature = [0, 1, 0, 0, 0, 0]
    what = ['Identity Number', 'Contract Number', 'Amount', 'Type', 'Term (month)', 'Annuity']
    stroka = sheet.nrows
    stolbec = sheet.ncols

    for i in range(stroka):
        for j in range(stolbec):
            for k in range(6):
                if sheet.cell(i, j).value == what[k]:
                    feature[k] = sheet.cell(i + 1, j).value


    # SQL
    sql = "insert into contract(id, contrnumb, amount, type, term, annuity) VALUES (?, ?, ?, ?, ?, ?)"
    cursor.execute(sql, (feature[0], feature[1], feature[2], feature[3], feature[4], feature[5]))

    sql4 = "commit"
    cursor.execute(sql4)
    print(f)

conn.commit()

#Отобразим таблицу с данными по контрактам
conn = sqlite3.connect("credit.db")
cursor = conn.cursor()
sql = "SELECT * FROM contract"
cursor.execute(sql)
for row in cursor.execute("SELECT rowid, * from contract"):
    print(row)


import sqlite3
#Объединим таблицы с данными анкет и контрактов по id
con = sqlite3.connect("credit.db")
with con:
    cursor = con.cursor()
    cursor.execute("SELECT * FROM anket INNER JOIN contract ON anket.id = contract.id")
    rows = cursor.fetchall()
for row in rows:
        print (row)

con.commit()

#Отобразим таблицу с данными по контрактам
con = sqlite3.connect("credit.db")
cursor = con.cursor()
sql = "SELECT * FROM anket"
cursor.execute(sql)
for row in cursor.execute("SELECT rowid, * from anket"):
    print(row)
con.commit()


#################    ВТОРОЙ ПУНКТ     #######################
########### ПРОВЕРКА АНКЕТ ############

con = sqlite3.connect("credit.db")
with con:
    cursor = con.cursor()

#Дата рождения должна быть меньше даты выпуска
cursor.execute("SELECT COUNT(*) FROM anket WHERE birth > issue")
anketerror1 = cursor.fetchone()[0]
rows = cursor.fetchall()
if anketerror1 > 0:
    cursor.execute("DELETE FROM anket WHERE birth > issue")
else:
    print("Birth data is correct")

#Количество детей не может быть отрицательным
cursor.execute("SELECT COUNT(*) FROM anket WHERE children < 0")
anketerror2 = cursor.fetchone()[0]
rows = cursor.fetchall()
if anketerror2 > 0:
    cursor.execute("DELETE FROM anket WHERE children < 0")
else:
    print("Children data is correct")

#Количество людей в семье не может быть отрицательным
cursor.execute("SELECT COUNT(*) FROM anket WHERE family < 0")
anketerror3 = cursor.fetchone()[0]
rows = cursor.fetchall()
if anketerror3 > 0:
    cursor.execute("DELETE FROM anket WHERE family < 0")
else:
    print("Family data is correct")

#Количество детей в семье меньше количества членов семьи. Если не так, то заменим значения в столбце family на кол-во детей в семье + 1
cursor.execute("SELECT COUNT(*) FROM anket WHERE family = children")
anketerror4 = cursor.fetchone()[0]
rows = cursor.fetchall()
if anketerror4 > 0:
    cursor.execute("UPDATE anket SET family = children + 1 WHERE  family = children")
else:
    print("Information about family is correct")

#Доход не может быть отрицательным
cursor.execute("SELECT COUNT(*) FROM anket WHERE income < 0")
anketerror5 = cursor.fetchone()[0]
rows = cursor.fetchall()
if anketerror5 > 0:
    cursor.execute("DELETE FROM anket WHERE income < 0")
else:
    print("Income data is correct")

#Предположим, если доход больше 100000, то у человека есть собственный дом/квартира. Если нет, то произведем замену значений housing для данных позиций.
cursor.execute("SELECT COUNT(*) FROM anket WHERE income > 100000 and housing != 'House / apartment'")
anketerror6 = cursor.fetchone()[0]
rows = cursor.fetchall()
if anketerror6 > 0:
    print("There is a strange position in housing") * anketerror6
    cursor.execute("UPDATE housing SET rating = 'House / apartment' WHERE  income > 100000 and housing != 'House / apartment'")
else:
    print("There are no strange positions in housing")

#Проверка на единство обозначений housing
cursor.execute("SELECT housing, COUNT(housing) FROM anket GROUP BY housing")
rows = cursor.fetchall()
for row in rows:
    print(row)


#Проверка на единство обозначений marital status
cursor.execute("SELECT mrtstatus, COUNT(mrtstatus) FROM anket GROUP BY mrtstatus")
rows = cursor.fetchall()
for row in rows:
    print(row)

cursor.execute("UPDATE anket SET mrtstatus = 'no data' WHERE  mrtstatus IS NULL")

#Проверка на единство обозначений gender
cursor.execute("SELECT gender, COUNT(gender) FROM anket GROUP BY gender")
rows = cursor.fetchall()
for row in rows:
    print(row)
cursor.execute("DELETE FROM anket WHERE gender IS NULL")

#Проверка на единство обозначений educ
cursor.execute("SELECT educ, COUNT(educ) FROM anket GROUP BY educ")
rows = cursor.fetchall()
for row in rows:
    print(row)

#Проверка на единство обозначений employ
cursor.execute("SELECT empl, COUNT(empl) FROM anket GROUP BY empl")
rows = cursor.fetchall()
for row in rows:
    print(row)





#################    ВТОРОЙ ПУНКТ     #######################
########### ПРОВЕРКА КОНТРАКТОВ ############

con = sqlite3.connect("credit.db")
with con:
    cursor = con.cursor()

#Сумма кредита - неотрицательная величина
cursor.execute("SELECT COUNT(*) FROM contract WHERE amount < 0")
contrerror1 = cursor.fetchone()[0]
rows = cursor.fetchall()
if contrerror1 > 0:
    print(rows)
    cursor.execute("DELETE FROM contract WHERE amount < 0")
if contrerror1 == 0:
    print("Amount data is correct")

#Сумма аннуитета - неотрицательная величина
cursor.execute("SELECT COUNT(*) FROM contract WHERE annuity < 0")
contrerror2 = cursor.fetchone()[0]
rows = cursor.fetchall()
if contrerror2 > 0:
    print(rows)
    cursor.execute("DELETE FROM contract WHERE annuity < 0")
if contrerror2 == 0:
    print("Annuity data is correct")

#Сумма аннуитета не должна превышать сумму кредита
cursor.execute("SELECT COUNT(*) FROM contract WHERE annuity > amount")
contrerror3 = cursor.fetchone()[0]
rows = cursor.fetchall()
if contrerror3 > 0:
    print(rows)
    cursor.execute("DELETE FROM contract WHERE annuity > amount")
if contrerror3 == 0:
    print("Annuity is less than Amount. It's ok")

#Срок кредита не может быть отрицательным
cursor.execute("SELECT COUNT(*) FROM contract WHERE term < 0")
contrerror4 = cursor.fetchone()[0]
rows = cursor.fetchall()
if contrerror4 > 0:
    print(rows)
    cursor.execute("DELETE FROM contract WHERE term < 0")
if contrerror4 == 0:
    print("Term data is correct")

#Сумма кредита не превышает общую сумму аннуитетных платежей
cursor.execute("SELECT COUNT(*) FROM contract WHERE amount > annuity * term")
contrerror5 = cursor.fetchone()[0]
rows = cursor.fetchall()
if contrerror5 > 0:
    print(rows)
    cursor.execute("DELETE FROM contract WHERE amount > annuity * term")
if contrerror5 == 0:
    print("Annuity sum is correct")

#Проверка на единство обозначений type
cursor.execute("SELECT type, COUNT(type) FROM contract GROUP BY type")
rows = cursor.fetchall()
for row in rows:
    print(row)





################     ТРЕТИЙ ПУНКТ     ###############
#Кодировка поля пол: Female - 0, Male - 1
cursor.execute("UPDATE anket SET gender = '0' WHERE gender = 'Female'")
cursor.execute("UPDATE anket SET gender = '1' WHERE gender = 'Male'")

#Кодировка поля образования: Secondary / secondary special - 1, Incomplete higher - 2, Higher education - 3
cursor.execute("UPDATE anket SET educ = '1' WHERE gender = 'Secondary / secondary special'")
cursor.execute("UPDATE anket SET educ = '2' WHERE gender = 'Incomplete higher'")
cursor.execute("UPDATE anket SET educ = '3' WHERE gender = 'Higher education'")

#Кодировка поля семейное положение:
cursor.execute("UPDATE anket SET mrtstatus = '0' WHERE mrtstatus = 'Separated' or mrtstatus = 'Single / not married' or mrtstatus = 'Widow'")
cursor.execute("UPDATE anket SET mrtstatus = '1' WHERE mrtstatus = 'Married' or mrtstatus = 'Civil marriage'")

#Кодировка поля семейное положение:
cursor.execute("UPDATE anket SET housing = '1' WHERE housing = 'House / apartment'")
cursor.execute("UPDATE anket SET housing = '0' WHERE housing = 'Municipal apartment' or housing = 'Rented apartment' or housing = 'With parents'")

#Кодировка поля тип займа:
cursor.execute("UPDATE contract SET type = '1' WHERE type = 'Cash loans'")
cursor.execute("UPDATE contract SET type = '0' WHERE type = 'Revolving loans'")



conn.commit()

#Создаем общую таблицу таблицу c
conn = sqlite3.connect("credit.db")
cursor = conn.cursor()
cursor.execute("""create table generaltable(id integer,
               birth real,
               children integer,
               family integer,
               income integer,
               issue real,
               gender integer,
               mrtstatus integer,
               educ integer,
               housing integer, 
               contrnumb text,
               amount integer,
               type integer,
               term integer,
               annuity integer)""")
cursor.execute("INSERT INTO generaltable(id, birth, children, family, income, issue, gender, mrtstatus, educ, housing, contrnumb, amount, type, term, annuity) SELECT id, birth, children, family, income, issue, gender, mrtstatus, educ, housing, contrnumb, amount, type, term, annuity FROM anket")
cursor = conn.cursor()
sql = "select * from generaltable"
cursor.execute(sql)
for row in cursor.execute("select rowid, * from generaltable"):
    print(row)

conn.commit()

#Отобразим общую (финальную) таблицу с данными по анкетам и контрактам
conn = sqlite3.connect("credit.db")
cursor = conn.cursor()
sql = "select * from generaltable"
cursor.execute(sql)
for row in cursor.execute("select rowid, * from generaltable"):
    print(row)

conn.commit()

