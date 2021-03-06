## Основы программирования в R
### Модуль 2: продвинутые структуры
#### Матрицы и списки  
> Матрица - двумерный массив данных одного типа; вектор, уложенный по столбцам  

`matrix` для создания матрицы :  
**m** <-matrix (1:6, nrow = 2, ncol = 3)  *#матрица с элементами от 1 до 6 с 2 строками и 3 столбцами*  
matrix (1:6, nrow = 2)               *#то же самое  (1 2)- первый столбец*  
matrix(1:6, nrow = 2, byrow = TRUE)  *#(1 2 3) - первая строка*  
**dim(m)** <- NULL #из матрицы в вектор  
**rownames(m)** <- c("row1", "row2")     *#дадим имена строчкам*  
**colnames(m)** <- paste0("column", 1:3) *#и столбцам*  
m["row1", c("column2", "column3"), drop = F]  *#исключить все кроме пересечений указанных столбцов и строки*  
rbind(m1, m2) и cbind(m1, m2)                 *#объединение матриц по строкам и столбцам соответственно*  

Создадим функцию,которая объединяет две матрицы (верхний левый угол - первая матрица, нижний правый - вторая) и заполняет остальные элементы определенными значениями:
```{r}
bind_diag <- function(m1, m2, fill) {
  m3 <- matrix(fill, 
               nrow = nrow(m1) + nrow(m2), 
               ncol = ncol(m1) + ncol(m2))
  m3[1:nrow(m1), 1:ncol(m1)] <- m1
  m3[nrow(m1) + 1:nrow(m2), ncol(m1) + 1:ncol(m2)] <- m2
  m3
}

m1 <- matrix(1:12, nrow = 3)
m2 <- matrix(10:15, ncol = 3)
bind_diag(m1, m2, fill = NA)
bind_diag(m2, m1, fill = 0)
```

**Задачи:**  
**1:** *Предположим, что у нас есть целочисленный вектор v и число n. Наша задача — найти позицию элемента в векторе, который ближе всего к числу n. 
При этом если таких элементов несколько, необходимо указать все позиции. Напишите функцию, которая принимает на вход вектор и число и возвращает вектор индексов, 
отвечающих указанному условию. Индексы должны быть выстроены по возрастанию.*
```{r}
find_closest <- function(v, n) {
    which(abs(v-n) == min(abs(v-n)))
}
```
**2:** *Напишите функцию, которая принимает одно целое число n, а возвращает “ступенчатую” матрицу, состоящую из n этажей. Этажи нумеруются с первого, ширина каждой ступеньки 
равна одной строке или столбцу. Подсказка: обратите внимание на случай n=1! Зиккурат в этом случае будет матрицей 1 на 1, высотой 1.*
```{r}
build_ziggurat <- function(n) {
    zicc <- matrix(NA, nrow=n, ncol=n)   #делаем квадратную матрицу
    zicc <- pmin(row(zicc),col(zicc))    #pmin возвращают вектор с минимальными из каждой пары row(zicc),col(zicc)
      if(n>1){
        zicc <- cbind(zicc, zicc[,(n-1):1])
        zicc <- rbind(zicc, zicc[(n-1):1,])
      }
    return(zicc)
}
```
**3:** *Пусть x -- целочисленный вектор. Напишите функцию, которая вернёт матрицу из двух строк. В первой строке перечислите все различные элементы вектора, упорядоченные по возрастанию. Во второй строке укажите частоты (количество повторов) этих элементов.*
```{r}

count_elements <- function(x) {
  return(rbind(
    sort(unique(x)),
    table(x)            #таблица сопряжения с количеством повторяющихся значений
  ))
}
```

***
#### Дата фреймы

> length(df) возвращает количество столбцов (переменных), а не общее кол-во элементов.

##### Фильтрация по условию

```{r}
df[df$x > 2, ] 
```

```{r}
subset(df, x > 2) #в subset не надо дублировать название датафрейма
```

```{r}
subset(df, x > 2, select = c(x, z)) #выбор только переменных x и z

**Задачи:**  
**1:** *Дата фрейм attitude -- встроенный массив данных, содержащий рейтинг департаментов одной финансовой компании, составленный сотрудниками. Представьте, что вы хотите устраиваться как раз в эту компанию, и дата фрейм (совершенно случайно!) оказался в вашем распоряжении. Вы решили, что самое главное для вас -- это возможность учиться новому (learning). Возьмите 5 топовых департаментов по этому показателю. Из этого набора вам более всего подойдёт тот департамент, который имеет наибольшую сумму баллов по трём показателям: реакция на жалобы работников (complaints), надбавки в зависимости от результатов работы (raises) и возможность продвижения (advance).
Какой же департамент вам выбрать? Напишите его номер XX (номер строки в дата фрейме).*

```{r}
top5 <- sort(attitude$learning, decreasing = T)[1:5] #топ-5 департаментов по learning

row_number <- which(attitude$learning >= min(top5)) #номера строк этих департаментов

new_table <- rbind(attitude[row_number,]) 

top_sums <- rowSums(subset(new_table, select = c(complaints,raises,advance))) #суммы по показателям complaints, raises и advance.

names(which.max(top_sums)) # имя максимального значения в векторе
}
```
> any(!**complete.cases**(df)) - есть ли пропуски в датафрейме


**2:** *...Cкачайте файл по ссылке, добавьте новые данные в общий дата фрейм и повторите подсчёт общего покрытия, добавив переменную total_coverage. В качестве ответа пришлите величину среднего покрытия с точностью до второго знака: X.XX*

```{r}
df1 <- read.csv('https://raw.githubusercontent.com/tonytonov/Rcourse/master/R%20programming/avianHabitat.csv')
df2 <- read.csv(sep = ";", skip = 5, header = T, comment.char = "%",
                quote = "", na.strings = "Don't remember",
                file = 'https://raw.githubusercontent.com/tonytonov/Rcourse/master/R%20programming/avianHabitat2.csv')
names(df1)
names(df2)
df2$Observer <- 'CL'
df2 <- df2[,c(1,17,2:16)]     # Переносим 17 столбец на место второго
newdf <- rbind(df1,df2)
head(newdf)
coverage_var <- names(newdf)[-(1:4)][c(T,F)]
newdf$total_coverage <- rowSums(newdf[,coverage_var])
mean(newdf$total_coverage)

```


