---
title: Оптимизация через разделяй-и-властвуй
weight: 1
prerequisites:
- .
---

*Эта статья — одна из [серии](../). Рекомендуется сначала прочитать все предыдущие.*

Посмотрим на формулу пересчета динамики для базового решения:

$$
f[i, j] = \min_{k < i} \{f[k, j-1] + (x_{i-1}-x_k)^2 \}
$$

Обозначим за $opt[i, j]$ оптимальный $k$ для данного состояния — то есть  от выражения выше. Для однозначности, если оптимальный индекс не один, то выберем среди них самый правый.

Конкретно в задаче покрытия точек отрезками, можно заметить следующее:

$$
opt[i, j] \leq opt[i, j+1]
$$

Интуиция такая: если у нас появился дополнительный отрезок, то последний отрезок нам не выгодно делать больше, а скорее наоборот его нужно «сжать».

### Идея

Пусть мы уже знаем $opt[i, l]$ и $opt[i, r]$ и хотим посчитать $opt[i, j]$ для какого-то $j$ между $l$ и $r$. Тогда, воспользовавшись неравенством выше, мы можем сузить отрезок поиска оптимального индекса для $j$ со всего отрезка $[0, i-1]$ до $[opt[i, l], opt[i, r]]$.

Будем делать следующее: заведем рекурсивную функцию, которая считает динамики для отрезка $[l, r]$, зная, что их $opt$ лежат между $l'$ и $r'$. Эта функция просто берет середину отрезка $[l, r]$ и линейным проходом считает ответ для неё, а затем рекурсивно запускается от половин, передавая в качестве границ $[l', opt]$ и $[opt, r']$ соответственно.

### Реализация

Один $k$-тый слой целиком пересчитывается из $(k-1)$-го следующим образом:

```c++
void solve(int l, int r, int _l, int _r, int k) {
    if (l > r)
        return; // отрезок пустой -- выходим
    int opt = _l, t = (l + r) / 2;
    for (int i = _l; i <= min(_r, t); i++) { 
        int val = f[i + 1][k - 1] + cost(i, j);
        if (val < f[t][k])
            f[t][k] = val, opt = i;
    }
    solve(l, t - 1, _l, opt, k);
    solve(t + 1, r, opt, _r, k);
}
```

Затем последовательно вызовем эту функцию для каждого слоя:

```c++
for (int k = 1; k <= m; k++)
    solve(0, n - 1, 0, n - 1, k);
```

### Асимптотика

Так как отрезок $[l, r]$ на каждом вызове уменьшается примерно в два раза, глубина рекурсии будет $O(\log n)$. Так как отрезки поиска для всех элементов на одном «уровне» могут пересекаться разве что только по границам, то суммарно на каждом уровне поиск проверит $O(n)$ различных индексов. Соответственно, пересчет всего слоя займет $O(n \log n)$ операций вместо $O(n^2)$ в базовом решении.

Таким образом, мы улучшили асимптотику до $O(n m \log n)$.
