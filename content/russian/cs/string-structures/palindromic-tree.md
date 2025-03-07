---
title: Дерево палиндромов
authors:
- Сергей Слотин
weight: 3
---

Дерево палиндромов (англ. *palindromic tree*, *EERTREE*) — структура данных, использующая другой, более мощный формат хранения информации обо всех подпалиндромах, чем размеры $n$ палиндромов. Она была предложена Михаилом Рубинчиком на летних петрозаводских сборах в 2014-м году.

**Лемма.** В строке есть не более $n$ различных подпалиндромов.

**Доказательство.** Пусть мы дописываем к строке по одному символу и в данный момент, записав $r$ символов, имеем наибольший суффикс-палиндром $s_{l:r}$. Пусть у него, в свою очередь, есть суффикс-палиндром $s_{l':r} = t$. Тогда он также имеет более раннее вхождение в строку как $s_{l:l+r-l'} = t$. Таким образом, с каждым новым символом у строки появляется не более одного нового палиндрома, и если таковой есть, то это всегда наибольший суффикс-палиндром.

Этот факт позволяет сопоставить всем палиндромам строки сопоставить следующую структуру: возьмём от каждого палиндрома его правую половину (например, $caba$ для $abacaba$ или $ba$ для $abba$; будем рассматривать пока что только чётные палиндромы) и добавим все эти половины в префиксное дерево — получившуюся структуру и будем называть *деревом палиндромов*.

Наивный алгоритм построения будет в худшем случае работать за $O(n^2)$, но это можно делать и более эффективно.

### Построение за линейное время

Будем поддерживать наибольший суффикс-палиндром. Когда мы будем дописывать очередной символ $c$, нужно найти наибольший суффикс этого палиндрома, который может быть дополнен символом $c$ — это и будет новый наидлиннейший суффикс-палиндром.

Для этого поступим аналогично [алгоритму Ахо-Корасик](aho-corasick): будем поддерживать для каждого палиндрома суффиксную ссылку $l(v)$, ведущую из $v$ в её наибольший суффикс-палиндром. При добавлении очередного символа, будем подниматься по суффиксным ссылкам, пока не найдём вершину, из которой можно совершить нужный переход.

Если в подходящей вершине этого перехода не существовало, то нужно создать новую вершину, и для неё тоже понадобится своя суффиксная ссылка. Чтобы найти её, будем продолжать подниматься по суффиксным ссылкам предыдущего суффикс-палиндрома, пока не найдём второе такое место, которое мы можем дополнить символом $c$.

```c++
const int maxn = 1e5, k = 26;

int s[maxn], len[maxn], link[maxn], to[maxn][k];

int n, last, sz;

void init() {
    s[n++] = -1;
    link[0] = 1;
    len[1] = -1;
    sz = 2;
}

int get_link(int v) {
    while (s[n-len[v]-2] != s[n-1])
        v = link[v];
    return v;
}

void add_char(int c) {
    s[n++] = c;
    last = get_link(last);
    if (!to[last][c]) {
        len[sz] = len[last] + 2;
        link[sz] = to[get_link(link[last])][c];
        to[last][c] = sz++;
    }
    last = to[last][c];
}
```

Здесь мы использовали обычный массив для хранения переходов. Как и для любых префиксных деревьев, вместо него можно использовать бинарное дерево поиска, хеш-таблицу, односвязный список и другие структуры, позволяющие обменять время на память, немного изменив асимптотику.

### Асимптотика

Покажем линейность алгоритма. Рассмотрим длину наибольшего суффикс-палиндрома строки. Каждый новый символ увеличивает её не более, чем на 2. При этом каждый переход по суффиксной ссылке уменьшает её, поэтому нахождение первого суффикс-палиндрома амортизировано работает за линейное время.

Аналогичными рассуждениями о длине второго суффикс-палиндрома (его длина увеличивается тоже не более, чем на 2) получаем, что пересчёт суффиксных ссылок при создании новых вершин тоже суммарно работает за линейное время.
