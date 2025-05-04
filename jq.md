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

Кроме того, мы можем настроить путь JSON для соответствия структуре данных. Также jq является хорошим выбором для обработки данных в высокопроизводительных средах. jq эффективно обрабатывает большие наборы данных, сокращая задержку сети и улучшая пользовательский опыт.

```bash
#!bin/sh
NAME_VACANCY="data+scientist"

 curl -o hh.json https://api.hh.ru/vacancies?text=$NAME_VACANCY
```

```bash
#!bin/sh

FILE_JSON=../ex00/hh.json
OUTPUT_CSV=hh.csv
FILTER=filter.jq
title="id,created_at,name,has_test,alternate_url"

 echo $title > $OUTPUT_CSV
 jq -r -f $FILTER $FILE_JSON >> $OUTPUT_CSV
```

```bash
#!bin/sh

FILE=../ex01/hh.csv
OUTPUT=hh_sorted.csv

head -1 $FILE > $OUTPUT
tail -20 $FILE | sort -t"," -k2 -k1  >> $OUTPUT

cat $OUTPUT
```
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
