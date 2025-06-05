# Assigment2_DB
## Кроки оптимізації
Запит підраховує к-ть студентів, що мають вищу оцінку за середню для статі та їх соціально-економічного статусу.
<img width="612" alt="Знімок екрана 2025-06-05 о 17 34 06" src="https://github.com/user-attachments/assets/7235a4c3-4d4f-4b58-80a6-7dfd07ef9e02" />

### Початковий запит:
  <img width="392" alt="Знімок екрана 2025-06-05 о 17 25 20" src="https://github.com/user-attachments/assets/8696e59e-2508-4814-9aa7-35fceeda9fb8" />
1.Першочергово можемо побачити, що повторюється фільтр, отже його треба винести як CTE
  <img width="832" alt="Знімок екрана 2025-06-05 о 17 26 01" src="https://github.com/user-attachments/assets/bafa6cb6-d32a-4577-9447-ed0b8e6c7525" />
2. Далі, так як, логіка присутня і немає зайвих підзапитів чи частин коду, встановлюємо індекс
  <img width="699" alt="Знімок екрана 2025-06-05 о 17 34 40" src="https://github.com/user-attachments/assets/722cf79a-87b7-436b-96a2-d1349e2570b3" />
  Я вирішила створити саме covered index, адже запит обирає невелику к-ть стовпців і якщо ми його створимо це тільки пришвидшить виконання, а не сповільнить, за рахунок принципу         роботи covered index.

## Порівняння EXECUTION PLAN
``` -> Table scan on <temporary>  (actual time=3079..3079 rows=6 loops=1)
    -> Aggregate using temporary table  (actual time=3079..3079 rows=6 loops=1)
        -> Filter: ((f.Age >= 16) and (f.SES_Quartile >= 2) and (f.TestScore_Math > '74.98159968928803'))  (cost=807018 rows=293352) (actual time=1085..2492 rows=1.8e+6 loops=1)
            -> Covering index scan on f using idx_filtering  (cost=807018 rows=7.92e+6) (actual time=0.192..2172 rows=8e+6 loops=1)
```
