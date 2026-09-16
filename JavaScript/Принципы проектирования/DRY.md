# ЧАСТЬ 2. DRY (Don't Repeat Yourself)

**Формулировка:** "Не повторяй себя" — каждая часть знания/логики должна иметь единственное, однозначное представление в системе. Если один и тот же код (или знание о том, как что-то работает) продублирован в нескольких местах — при изменении придётся не забыть поправить **все** копии, что рано или поздно приводит к рассинхронизации и багам.

### ❌ Плохо — одинаковая логика скопирована в нескольких местах

```javascript
function calculateOrderTotal(price, quantity) {
  const subtotal = price * quantity;
  const tax = subtotal * 0.2; // ставка налога "зашита" здесь
  return subtotal + tax;
}

function calculateInvoiceTotal(price, quantity) {
  const subtotal = price * quantity;
  const tax = subtotal * 0.2; // и здесь тоже — дублирование!
  return subtotal + tax;
}

function calculateRefund(price, quantity) {
  const subtotal = price * quantity;
  const tax = subtotal * 0.2; // и здесь — тройное дублирование
  return subtotal + tax;
}
```

**Проблема:** если налоговая ставка изменится (`0.2` → `0.25`), нужно вспомнить и поправить **все три** функции. Если забыть хотя бы одну — возникнет трудноуловимый баг: в разных частях системы будут разные расчёты.

### ✅ Хорошо — логика вынесена в одно место

```javascript
const TAX_RATE = 0.2; // единственный источник истины

function calculateTotalWithTax(price, quantity) {
  const subtotal = price * quantity;
  const tax = subtotal * TAX_RATE;
  return subtotal + tax;
}

function calculateOrderTotal(price, quantity) {
  return calculateTotalWithTax(price, quantity);
}

function calculateInvoiceTotal(price, quantity) {
  return calculateTotalWithTax(price, quantity);
}

function calculateRefund(price, quantity) {
  return calculateTotalWithTax(price, quantity);
}
```

Теперь изменение ставки налога (`TAX_RATE`) или самой формулы расчёта происходит **в одном месте** и автоматически отражается везде, где эта функция используется.

**Важный нюанс DRY:** принцип не про "никогда не писать похожий код", а про **дублирование знания**. Иногда два похожих на вид куска кода решают на самом деле разные задачи, которые просто _случайно_ выглядят одинаково сейчас — их слияние в одну функцию может привести к неправильной, излишне усложнённой абстракции, если задачи начнут расходиться позже. Здесь важно различать "случайное совпадение" и "настоящее дублирование одного и того же знания".