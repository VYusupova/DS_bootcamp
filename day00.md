## Упражнение 00 : Первый сценарий командной строки  

Файлы для передачи: `hh.sh`, `hh.json`  
Разрешенные функции : `curl`, `jq` 

В этом упражнении вам нужно будет взаимодействовать с **API HeadHunter**, чтобы получить некоторую информацию о вакансиях. Для этого вам нужно понять, как работают `curl` и *API HeadHunter* - https://dev.hh.ru/.

Напишите сценарий оболочки, который:  
в качестве аргумента получает название вакансии —  **data scientist**  (некоторые последующие упражнения будут основаны на этом), загружает информацию о первых 20 вакансиях, соответствующих параметрам поиска,
сохраняет его в файле с именем `hh.json`.
Результат в файле должен быть отформатирован таким образом, чтобы каждое поле находилось на отдельной строке.  

Решение:
```bash
#!bin/sh
NAME_VACANCY="" #"data+scientist"

if [ -n "$1" ]
then
NAME_VACANCY=$1
    if [ -n "$2" ]
    then
    NAME_VACANCY=$NAME_VACANCY+$2
    fi
else
echo "No parameters found. "
fi
echo $NAME_VACANCY

 curl -o hh.json https://api.hh.ru/vacancies?text=$NAME_VACANCY

```
## Упражнение 01 : Преобразование JSON в CSV  

Файлы для отправки: filter.jq, json_to_csv.sh, hh.csv  
Разрешенные функции : jq  

То, что вы получили в предыдущем упражнении, было файлом JSON.  
Это популярный формат файлов для API, но он может быть неудобен для анализа данных.  
Поэтому вам нужно будет преобразовать его в более удобный файл CSV.

Напишите сценарий оболочки под названием *json_to_csv.sh* который:  
выполняет jq с фильтром, записанным в отдельный файл filter.jq  
фильтрует следующие 5 столбцов, соответствующих вакансиям: «id», «created_at», «name», «has_test» и «alternate_url»  
сохраняет результат в CSV-файл hh.csv  

Теория:  
**jq** — это простой и мощный командный процессор JSON в Linux.  
Фактически, он позволяет анализировать, фильтровать и манипулировать данными JSON.  
Давайте использовать приведенный ниже файл data.json в качестве входного файла в этой статье:
```
$ cat data.json
{
    "employees": [
        {
            "name": "Alice",
            "department": "HR",
            "age": 28
        },
        {
            "name": "Bob",
            "department": "IT",
            "age": 32
        }
    ]
}
```

Теперь давайте воспользуемся jq для преобразования JSON в CSV:  
`$ jq -r '.employees[] | [.name, .department, .age] | @csv' data.json > data.csv`

Здесь,

`jq -r ` указывает jq работать в «сыром» режиме , что гарантирует пригодность вывода для форматирования CSV.  
`[.name, .department, .age] | Фильтр @csv` обрабатывает правильное экранирование значений и преобразует данные JSON в формат CSV  
Затем сохраняем полученный CSV-файл как data.csv :  
```
$ cat data.csv
name,department,age
Alice,HR,28
Bob,IT,32
```

Решение:  (!добавить фильтр)
```bash
#!bin/sh

FILE_JSON=../ex00/hh.json
OUTPUT_CSV=hh.csv
FILTER=filter.jq
title="id,created_at,name,has_test,alternate_url"

 echo $title > $OUTPUT_CSV
 jq -r -f $FILTER $FILE_JSON >> $OUTPUT_CSV
```


## Упражнение 02 : Сортировка файла  

Файлы для передачи: sorter.sh, hh_sorted.csv
Разрешенные функции : cat, sort, head, tail

Напишите сценарий оболочки под названием sorter.sh который:

Отсортируйте файл hh.csv из предыдущего упражнения по столбцу «created_at», а затем по «id» в порядке возрастания  
сохраняет результат в CSV-файле hh_sorted.csv  
CSV-файл по-прежнему должен содержать заголовки в первой строке.  


Решение: 
```bash
#!bin/sh

FILE=../ex01/hh.csv
OUTPUT=hh_sorted.csv

head -1 $FILE > $OUTPUT
tail -20 $FILE | sort -t"," -k2 -k1  >> $OUTPUT

cat $OUTPUT
```

## Упражнение 03 : Замена строк в файле  

Файлы для передачи: cleaner.sh, hh_positions.csv
Разрешенные функции : без ограничений

Необработанные данные — это беспорядок. Прежде чем приступить к их анализу, необходимо выполнить большую часть предварительной обработки. В этом упражнении вы продолжаете это делать. Если вы посмотрите на свой файл из предыдущего упражнения, то увидите, что название каждой должности содержит «Data Scientist» (вам не нужно это проверять). Это неудивительно, поскольку мы использовали эту строку в качестве ключевого слова для поиска в API HeadHunter. Но для нас, как и для алгоритмов, это не даёт никакой полезной информации. Честно говоря, это просто шум, который ухудшает анализ данных.

Напишите оболочку под названием script cleaner.sh которая:

убирает “Junior”, “Middle”, “Senior” из названий должностей, если название не содержит ни одного из этих слов, используйте «-» (“Senior Data Scientist” -> “Senior”, “analyst /(data scientist)” -> “-”, “Специалист / data scientist (big data, прогностическая аналитика, data mining)” -> “-” )), если их несколько, сохраните все (например, “Middle/Senior Data Scientist” -> “Middle/Senior”).
сохраняет результат в CSV-файле hh_positions.csv.  
В CSV-файле по-прежнему должны быть заголовки в первой строке, и он должен быть отсортирован в соответствии с предыдущим заданием.  

Теория:
https://habr.com/ru/companies/ruvds/articles/327754/  
-F fs — позволяет указать символ-разделитель для полей в записи.  
-f file — указывает имя файла, из которого нужно прочесть awk-скрипт.  
-v var=value — позволяет объявить переменную и задать её значение по умолчанию, которое будет использовать awk.  
-mf N — задаёт максимальное число полей для обработки в файле данных.  
-mr N — задаёт максимальный размер записи в файле данных.  
-W keyword — позволяет задать режим совместимости или уровень выдачи предупреждений awk.  

```bash
echo | awk -v home=$HOME '{print "My home is " home}'
```

Решение:   
```bash
#!bin/sh

FILE=../ex02/hh_sorted.csv
OUTPUT=hh_positions.csv
head -1 $FILE > $OUTPUT
tail -20 $FILE |  awk -F',' '{
            if ($3 ~ /Junior/)  print $1","$2", Junior ,"$4","$5;
            else
             if ($3 ~ /Middle/)  print $1","$2",Middle ,"$4","$5;
             else
             if ($3 ~ /Senior/)  print $1","$2",Senior ,"$4","$5;
             else print $1","$2", ,"$4","$5;
             }'  >> hh_positions.csv

```
```bash
#!bin/sh

FILE=../ex03/hh_positions.csv
OUTPUT=hh_uniq_positions.csv
tmp=tmp.csv

JUN=$(grep -c 'Junior' $FILE)
echo "Junior,"$JUN > $tmp

MID=$(grep -c 'Middle' $FILE)
echo "Middle,"$MID >> $tmp


SEN=$(grep -c 'Senior' $FILE)
echo "Senior,"$SEN >> $tmp

echo "name,count" > $OUTPUT
sort $tmp -rt"," -k2   >> $OUTPUT

rm -rf $tmp
cat $OUTPUT

```
```bash
#!bin/sh

FILE=../ex03/hh_positions.csv
OUTPUT=hh_uniq_positions.csv
tmp=tmp.csv
header=$(head -1 $FILE)

tail -20 $FILE > $tmp

exec 0< $tmp

date="1"
while read line
do
ndate=${line:10:10}
if [[ $date -ne $ndate ]]
then
echo $header > $ndate.csv
echo $line >>  $ndate.csv

date=$ndate
else
echo $line >>  $date.csv

fi
done
echo $header

rm -rf $tmp

```

##  Упражнение 05. Разделение и конкатенация
Каталог для сдачи: `ex05/`
Файлы для отправки: `partitioner.sh`, `concatenator.sh`
Разрешенные функции : без ограничений

Если у вас большой набор данных, иногда бывает полезно разделить его на разделы. Каждый раздел имеет определённый диапазон ключей. Один из популярных способов разделения — по дате. Каждый раздел содержит данные за определённую дату. В этом упражнении вам нужно будет выполнить эту задачу.
Напишите сценарий оболочки под названием partitioner.sh, который: принимает на вход результат выполнения упражнения 03 сохраняет фрагменты данных с разными датами `created_at` в отдельных CSV-файлах, названных в соответствии с этой датой

Напишите ещё один сценарий оболочки под названием `concatenator.sh`, который: принимает на вход отдельные файлы, полученные в результате работы `partitioner.sh`
объединяет все отдельные файлы в один CSV-файл

В файлах CSV в первой строке должны быть заголовки, как в примере. CSV-файл, полученный в результате работы `concatenator.sh`, должен совпадать с результатом выполнения упражнения 3
Ваши скрипты должны быть исполняемыми. Используйте интерпретатор `/bin/sh`.

**partitioner.sh**
```bash
#!bin/sh
GREEN='\e[1;36;108m'
FILE=../ex03/hh_positions.csv
tmp=tmp.csv
header=$(head -1 $FILE)
tail -20 $FILE > $tmp
exec 0< $tmp
read line
date=`echo $line | cut -d, -f 2 | cut -dT -f 1`
echo $header > $date.csv
echo $line >>  $date.csv
while read line
do
ndate=`echo $line | cut -d, -f 2 | cut -dT -f 1` # ${line:10:10} #альтернативный вариант
if [ $date != $ndate ]
then
echo $header > $ndate.csv
echo $line >>  $ndate.csv
date=$ndate
else
echo $line >>  $date.csv
fi
done
printf "$GREEN ✅ partitioner.sh Execution completed successfully \e[31;0m \n"
rm -rf $tmp
```

**concatenator.sh**
```bash
#!bin/sh
RED='\e[1;31;108m'
GREEN='\e[1;36;108m'
def='\e[31;0m \n'
tmp=tmp.csv
FILE=../ex03/hh_positions.csv
OUTFILE=concat_file.csv
cat `ls *.csv` > $tmp
STR=`head -1 $tmp`
sed -i "/$STR/d" "$tmp"
 echo $STR > $OUTFILE
 cat $tmp >> $OUTFILE
rm -rf $tmp
printf "$GREEN ✅ concatenator.sh Execution completed successfully $def"
if cmp -s $FILE $OUTFILE ; then
    printf "$GREEN ✅ success \n\t$FILE = $OUTFILE $def";
    else
    printf "$RED 💥 fail \n\t$FILE <> $OUTFILE $def";
fi
```
