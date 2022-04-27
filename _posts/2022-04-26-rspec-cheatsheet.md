---
layout: post
title: "Yet another rspec cheatsheet"
---

Краткая выжимка неплохого [курса по тестированию](https://www.udemy.com/course/testing-ruby-with-rspec/) в свободной интерпретации и личными дополнениями. Этот cheatsheet содержит базовый функционал по работе с основными методами библиотеки rspec.  

Изначально это был мини курс, который назывался "Rspec простым языком". Он выдавался джунам, которых нужно было минимально научить пользоваться рспеком. После прочтения также выдавалить тестовые задания на закрепление.

Тестовые задания я решил убрать и выложить эту простую и супер-краткую версию курса как cheatsheet по rspec :)

# Базовый функционал
## Базовый пример теста
Тестируемый файл:
```ruby
class Card
  attr_accessor :rank, :suit
  def initialize(rank, suit)
    @rank = rank
    @suit = suit
  end
end
```

файл с тестами:
```ruby
RSpec.describe Card do
  let(:card) { Card.new('Ace', 'Spades') }

  it 'has a rank and that rank can change' do
    expect(card.rank).to eq('Ace')
    card.rank = 'Queen'
    expect(card.rank).to eq('Queen')
  end

  it 'has a suit' do
    expect(card.suit).to eq('Spades')
  end

  it 'has a custom error message' do
    card.suit = 'Nonsense'
    comparison = 'Spades'
    expect(card.suit).to eq(comparison), "Я ждал #{comparison} но получил #{card.suit}!"
  end
end
```
## describe
Метод Rpsec, который говорит что внутри него будет тестироваться какой-то объект нашей ситемы, в данном случае класс Card.
## it
Конкретный тест, проверяющий описанный далее сценарий. Этот блок называется example, в будущем я буду называть эти блоки тестами.  

В примере выше первый блок it проверяет что у объекта card, который принадлежит классу Card, есть метод rank и его можно назначать и читать. 
В примере у нас есть три теста(example), каждый из которых проверяет отдельный функционал.

Каждый из блоков должен быть независимым от других, т.е. не должно быть ситуации, в которой успешность теста зависит от порядка его запуска.
## let
Переменная в мире rspec. Особенность ее в том, что она пересоздается перед стартом каждого example(it блок). Поэтому если в каком-то блоке с ней произошли какие-то изменения, то это не будет влиять на другие блоки.  
В примере выше создается переменная card и она используется в каждом тесте.
## expect
Основной метод библиотеки. Используется для заключения что тест прошел успешно или неуспешно.  
Более простой пример:
```ruby
RSpec.describe Integer do
  it 'should pass' do
    expect(1 + 1).to eq(2)
  end
end
```
Результат запуска этого теста в консоли:
```
.

Finished in 0.00335 seconds (files took 0.12793 seconds to load)
1 example, 0 failures
```
В этом ответе мы видим что прошел один тест, ошибок не было.

Если поменять на ложную проверку
```ruby
RSpec.describe Integer do
  it 'should not pass' do
    expect(1 + 1).to eq(3)
  end
end
```
Результат:
```
F

Failures:

  1) Integer should pass
     Failure/Error: expect(1 + 1).to eq(3)
     
       expected: 3
            got: 2
     
       (compared using ==)
     # ./test.rb:3:in `block (2 levels) in <top (required)>'

Finished in 0.0232 seconds (files took 0.10606 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./test.rb:2 # Integer should pass
```
В этом результате мы видим, что всего 1 example и 1 фейл. Также написано что по тесту мы ждали, что там будет значение 3, но (сюрприз!) 1+1 оказалось равно двум, поэтому наш тест упал.
На этом методе базируются все проверки.

## subject
Прединициализированный объект, который является инстансом описываемого объекта.
Пример неявного(implicit) использования:
```ruby
RSpec.describe Hash do
  it 'should start off empty' do
    expect(subject.length).to eq(0)
    subject[:some_key] = "Some Value"
    expect(subject.length).to eq(1)
  end

  it 'is isolated between examples' do
    expect(subject.length).to eq(0)
  end
end
```
В примере выше мы описываем класс `Hash`, значит по всех тестах ниже мы можем обращаться к объекту `subject`, который является результатом `Hash.new`. 
Если бы в верхнем блоке было`RSpec.describe MyClass`, то в `subject` хранился бы результат `MyClass.new`

Пример явного(explicit) использования:
```ruby
RSpec.describe Hash do
  subject(:bob) do
    { a: 1, b: 2 }
  end

  it 'has two key-value pairs' do
    expect(subject.length).to eq(2)
    expect(bob.length).to eq(2)
  end

  describe 'nexted example' do
    it 'has two key-value pairs' do
      expect(subject.length).to eq(2)
      expect(bob.length).to eq(2)
    end
  end
end
```
Как видно из примера выше, содержимое `subject` можно указывать явно.
# Hooks
Регулярно в тестах встречаются моменты, когда перед каждым тестом нужно выполнить какую-то функцию или набор функций. Либо же после каджого теста. Для таких моментов существуют хуки. Они позволяют сокращать дубликацию кода и не нарушать [DRY](https://ru.wikipedia.org/wiki/Don%E2%80%99t_repeat_yourself).
Основные хуки:
- before(:context)  
Выполняется один раз перед всеми тестами в этом блоке describe
- after(:context)  
Выполняется один раз после всех тестов в этом блоке describe
- before(:example)  
Выполняется каждый раз перед каждым тестом в этом блоке describe
- after(:example)  
Выполняется каждый раз после каждого теста в этом блоке describe

Пример
```ruby
RSpec.describe 'before and after hooks' do
  before(:context) do
    puts 'Before context'
  end

  after(:context) do
    puts 'After context'
  end

  before(:example) do
    puts 'Before example'
  end

  after(:example) do
    puts 'After example'
  end

  it 'is just a random example' do
    expect(5 * 4).to eq(20)
  end

  it 'is just another random example' do
    expect(3 - 2).to eq(1)
  end
end
```
Результат запуска:
```
Before context
Before example
After example
.Before example
After example
.After context


Finished in 0.00352 seconds (files took 0.18995 seconds to load)
2 examples, 0 failures
```

# Context
Блок, позволяющий описать в каком контексте мы находимся. Упрощает чтение тестов, позволяет группировать тесты вместе по общему контексту
Пример:
```ruby
RSpec.describe '#even? method' do
  context 'with even number' do
    it 'should return true' do
      expect(4.even?).to eq(true)
    end
  end

  context 'with odd number' do
    it 'should return false' do
      expect(5.even?).to eq(false)
    end
  end
end
```
Функционально не влияет ни на что. Позволяет для человека более ясно описать контекст, в котором происходит тест

# Matchers
Матчеры, которые используются для сравнения значений в `expect` выражениях. Позволяют гибко проверять успешность прохождения теста.

## not_to
В общем-то противоположность метода `.to`
Пример:
```ruby
RSpec.describe 'not_to method' do
  it 'checks for the inverse of a matcher' do
    expect(5).not_to eq(10)
    expect([1, 2, 3]).not_to equal([1, 2, 3])
    expect(10).not_to be_odd
    expect([1, 2, 3]).not_to be_empty

    expect(nil).not_to be_truthy

    expect('Philadelphia').not_to start_with('car')
    expect('Philadelphia').not_to end_with('city')

    expect(5).not_to respond_to(:length)

    expect([:a, :b, :c]).not_to include(:d)

    expect { 11 / 3 }.not_to raise_error
  end
end
```

## eq vs eql vs equal
Так называемые equality matchers. Есть целых три варианта для сравнения объектов, но важно знать их разницу:
- eq  
  Под капотом сравнивает через знак ==. Проверяет совпадают ли значения объектов. Если объекты разные(int и float например), то он их скастит к одному общему типу
- eql  
  Проверяет равенство значений объектов, но не пытается их скастить к общему типу, если их типы отличаются.
- equal  
  Проверяет равенство объектов, т.е. он проверяет являются ли проверяемые объекты одним и тем же объектом(одинаковый object_id)

Пример:
```ruby
RSpec.describe 'equality matchers' do
  let(:a) { 3.0 }
  let(:b) { 3 }

  describe 'eq matcher' do
    it 'tests for value and ignores type' do
      expect(a).to eq(3)
      expect(b).to eq(3.0)
      expect(a).to eq(b)
    end
  end

  describe 'eql matcher' do
    it 'tests for value, including same type' do
      expect(a).not_to eql(3)
      expect(b).not_to eql(3.0)
      expect(a).not_to eql(b)

      expect(a).to eql(3.0)
      expect(b).to eql(3)
    end
  end

  describe 'equal and be matcher' do
    let(:c) { [1, 2, 3] }
    let(:d) { [1, 2, 3] }
    let(:e) { c }

    it 'cares about object identity' do
      expect(c).to eq(d)
      expect(c).to eql(d)

      expect(c).to equal(e)

      expect(c).not_to equal(d)
      expect(c).not_to equal([1, 2, 3])

      expect(:my_symbol).to equal(:my_symbol)
      expect('my string').not_to equal('my string')
    end
  end
end
```
Все тесты в этом примере пройдут успешно.  
Обрати внимание на строку `expect('my string').not_to equal('my string')`, когда мы создаем строку таким образом, это будут разные объекты. Поэтому матчер equal этого не простит.  

Но в то же время строка `expect(:my_symbol).to equal(:my_symbol)` спокойно пройдет. Потому что Symbol такой класс, который по своим характеристикам больше похож на Integer, чем на String, он иммутабельный и каждый symbol уникален. Каждый раз, когда вы видите в коде `:abc`, это один и тот же объект.  

Более подробно прочитать про Symbol'ы в руби можно в этой [статье](https://dmitrytsepelev.dev/why-has-ruby-symbols).  

Краткая разница String и Symbol:
```
irb(main):001:0> puts "string".object_id
47217136322240
=> nil
irb(main):002:0> puts "string".object_id
47217134571480
=> nil
irb(main):003:0> puts :symbol.object_id
885148
=> nil
irb(main):004:0> puts :symbol.object_id
885148
=> nil
```
## Семейство be методов
Пример:
```ruby
RSpec.describe 'be matchers' do
  it 'can test for truthiness' do
    expect(true).to be_truthy
    expect('Hello').to be_truthy
    expect(5).to be_truthy
    expect(0).to be_truthy
    expect(-1).to be_truthy
    expect(3.14).to be_truthy
    expect([]).to be_truthy
    expect([1, 2]).to be_truthy
    expect({}).to be_truthy
    expect(:symbol).to be_truthy
  end

  it 'can test for falsiness' do
    expect(false).to be_falsy
    expect(nil).to be_falsy
  end

  it 'can test for nil' do
    expect(nil).to be_nil

    my_hash = { a: 5 }
    expect(my_hash[:b]).to be_nil
  end
  
  it 'can be tested with predicate matchers' do
    expect(16 / 2).to be_even
    expect(15).to be_odd
    expect(0).to be_zero
    expect([]).to be_empty
  end
end
```
`be_truthy` - падает если объект является nil или false, во всех остальных случаях проходит успешно.  
`be_falsy` - проходит если объект nil или false  
`be_nil` - проходит, если объект является nil  
`be` - без параметров то же самое что и be_truthy  
`be_even` - объект должен быть четным  
`be_odd` - объект должен быть нечетным  

Семейство этих методов еще раз показывает сахарность руби :)
## Методы сравнения(comparison)
Иногда нужно сравнить объект с другим объектом, чтобы проверить успешность теста  
Пример:
```ruby
RSpec.describe 'comparison matchers' do
  it 'allows for comparison with built-in Ruby operators' do
    expect(10).to be > 5
    expect(8).to be < 15

    expect(1).to be >= -1
    expect(1).to be >= 1

    expect(22).to be <= 100
    expect(22).to be <= 22
  end

  describe 100 do
    it { is_expected.to be > 90 }
    it { is_expected.to be >= 100 }
    it { is_expected.to be < 500 }
    it { is_expected.to be <= 100 }
    it { is_expected.not_to be > 105 }
  end
end
```
В блоке `describe 100 do` появляется сокращенная(one-line) версия обычных блоков `it`. В данном случае `is_expected` то же самое, что и `expect(100)`. 

## all matcher
Проверяет, что все объекты в тестируемом объекте соотвествуют заданному условию  
Пример:
```ruby
RSpec.describe 'all matcher' do
  it 'allows for aggregate checks' do
    expect([5, 7, 9, 13]).to all(be_odd)
    expect([4, 6, 8, 10]).to all(be_even)
    expect([[], [], []]).to all(be_empty)
    expect([0, 0]).to all(be_zero)
    expect([5, 7, 9]).to all(be < 10)
  end

  describe [5, 7, 9] do
    it { is_expected.to all(be_odd) }
    it { is_expected.to all(be < 10) }
  end
end
```
Более красивая замена циклу с expect'ами.
## change matcher
Матчер, который проверяет состояние объекта
```ruby
RSpec.describe 'change matcher' do
  subject { [1, 2, 3, 4] }

  it 'checks that a method changes object state' do
    expect { subject.push(4) }.to change { subject.length }.by(1)
  end

  it 'accepts negative arguments' do
    expect { subject.pop }.to change { subject.length }.from(4).to(3)
    expect { subject.pop }.to change { subject.length }.by(-1)
  end
end
```
В первом тесте мы пихаем в subject новый элемент и проверяем, что после этого длинна объекта поменялась. На этом примере может показаться, что это бесполезный тест, но этот метод может понадобиться для проверки более сложных итерабельный структур.
### start_with, end_with matchers
Проверяют, что объект начинается или завершается чем-то
```ruby
RSpec.describe 'start_with and end_with matchers' do
  describe 'caterpillar' do
    it 'should check for substring at the beginning or end' do
      expect(subject).to start_with('cat')
      expect(subject).to end_with('pillar')
    end

    it { is_expected.to start_with('cat') }
    it { is_expected.to end_with('pillar') }
  end

  describe [:a, :b, :c, :d] do
    it 'should check for elements at beginning or end of the array' do
      expect(subject).to start_with(:a)
      expect(subject).to start_with(:a, :b)
      expect(subject).to start_with(:a, :b, :c)
      expect(subject).to end_with(:d)
      expect(subject).to end_with(:c, :d)
    end

    it { is_expected.to start_with(:a, :b) }
  end
end
```
## have_attributes
Матчер, который проверяет что переданный объект имеет определенные аттрибуты
```ruby
class ProfessionalWrestler
  attr_reader :name, :finishing_move

  def initialize(name, finishing_move)
    @name = name
    @finishing_move = finishing_move
  end
end

RSpec.describe 'have_attributes matcher' do
  describe ProfessionalWrestler.new('Stone Cold Steve Austin', 'Stunner') do
    it 'checks for object attribute and proper values' do
      expect(subject).to have_attributes(name: 'Stone Cold Steve Austin')
      expect(subject).to have_attributes(name: 'Stone Cold Steve Austin', finishing_move: 'Stunner')
    end

    it { is_expected.to have_attributes(name: 'Stone Cold Steve Austin') }
    it { is_expected.to have_attributes(name: 'Stone Cold Steve Austin', finishing_move: 'Stunner') }
  end
end
```
Для простоты я указал класс и тест в одном файле.

## include
Проверяет, что объект содержит другой объект
```ruby
RSpec.describe 'include matcher' do
  describe 'hot chocolate' do
    it 'checks for substring inclusion' do
      expect(subject).to include('hot')
      expect(subject).to include('choc')
      expect(subject).to include('late')
    end

    it { is_expected.to include('choc') }
  end

  describe [10, 20, 30] do
    it 'checks for inclusion in the array, regardless of order' do
      expect(subject).to include(10)
      expect(subject).to include(10, 20)
      expect(subject).to include(30, 20)
    end

    it { is_expected.to include(20, 30, 10) }
  end

  describe ({ a: 2, b: 4 }) do
    it 'can check for key existence' do
      expect(subject).to include(:a)
      expect(subject).to include(:a, :b)
      expect(subject).to include(:b, :a)
    end

    it 'can check for key-value pair' do
      expect(subject).to include(a: 2)
    end

    it { is_expected.to include(:b) }
    it { is_expected.to include(b: 4) }
  end
end
```
## raise_error
Более интересный матчер, проверяет что блок слева рейсает определенную ошибку
```ruby
def some_method
  x
end

class CustomError < StandardError; end

RSpec.describe 'raise_error matcher' do
  it 'can check for a specific error being raised' do
    expect { some_method }.to raise_error(NameError) # undefined local variable or method `x`
    expect { 10 / 0 }.to raise_error(ZeroDivisionError)
  end

  it 'can check for a user-created error' do
    expect { raise CustomError }.to raise_error(CustomError)
  end
end
```
Полезно при проверке каких-то критичных значений, которые должны рейсать ошибки. При этом важно, что можно проверить какую именно ошибку рейсает блок из expect

## respond_to
Проверяет имеет ли переданный объект какой-то метод
```ruby
 class HotChocolate
  def drink
    'Delicious'
  end

  def discard
    'PLOP!'
  end

  def purchase(number)
    "Awesome, I just purchased #{number} more hot chocolate beverages!"
  end
end

RSpec.describe HotChocolate do
  it 'confirms that an object can respond to a method' do
    expect(subject).to respond_to(:drink)
    expect(subject).to respond_to(:drink, :discard)
    expect(subject).to respond_to(:drink, :discard, :purchase)
  end

  it 'confirms an object can respond to a method with arguments' do
    expect(subject).to respond_to(:purchase)
    expect(subject).to respond_to(:purchase).with(1).arguments
  end

  it { is_expected.to respond(:purchase, :discard) }
  it { is_expected.to respond(:purchase).with(1).arguments }
end
```
`expect(subject).to respond_to(:drink)` - тест пройдет, если `subject.drink` определен.

## Несколько expect'ов (Compound expectations)
Если нужно проверять сразу несколько условий после `expect`а
```ruby
RSpec.describe 25 do
  it 'can test for multiple matchers' do
    expect(subject).to be_odd.and be > 20
  end
end

RSpec.describe 'caterpillar' do
  it 'supports multiple matchers' do
    expect(subject).to start_with('cat').and end_with('pillar')
  end

  it { is_expected.to start_with('cat').and end_with('pillar') }
end

RSpec.describe [:usa, :canada, :mexico] do
  it 'can check for several possibilities' do
    expect(subject.sample).to eq(:usa).or eq(:canada).or eq(:mexico)
  end
end

```
Логика такая же, как и везде, с той лишь разницей, что тут это задается через `.and` и `.to`

# Mocks
Моки это всеобразные инструменты, позволяющие имитировать взаимодействие с чем-либо.  
Например в тесте проверяется апи с каким-то внешним сервисом, но нельзя каждый раз при запуске тестов реально туда обращаться. Для этого есть моки, которые, например, позволют имитировать http запросы куда-то. Т.е. в коде они делаются, но в реальности никуда запрос не летит и возвращает заранее заготовленный респонс.  То же самое можно делать с функциями, классами и чем угодно.

## double(двойник)
double(дабл) позволяет имитировать любой объект, задавая его поведение динамически
```ruby
RSpec.describe 'a random double' do
  it 'only allows defined methods to be invoked' do
    stuntman = double("Mr. Danger")
    allow(stuntman).to receive_messages(fall_off_ladder: 'Ouch', light_on_fire: true)
    expect(stuntman.fall_off_ladder).to eq('Ouch')
    expect(stuntman.light_on_fire).to eq(true)
  end
end
```
На 3 строке мы объявляем пустой дабл, строка `"Mr. Danger"` просто для идентификации и описания что это за дабл. На 4й строке методом `allow` мы задаем, что в дабле `stuntman` есть 2 метода: `fall_off_ladder`, который возвращает строку `Ouch` и метод `light_on_fire`, который возвращает `true`

*На самом деле правильно в руби говорить, что объект получает сообщение(поэтому метод и называется `receive_messages`), но для простоты я говорю, что этот дабл имеет методы `fall_off_ladder` и `light_on_fire`.*

Соответственно в expect'ах на 5 и 6 строках и проверяется, что дабл реализует два вышеописанных метода.

Использование двойников очень полезно, когда нам нужно изолировать тест, не делая его слишком зависимым от других систем. Например нам нужно изолированно протестировать класс А, который взаимодействует с классами В и С. Разумной идеей может быть замена взаимодействий с классами В и С через двойников, потому что когда мы проверяем исключительно класс А, мы не должны быть зависимыми от поломок в классах В и С(речь про изолированные тесты, а не про тестирование взаимодействия между этими классами).

## receive counts
Можно проверять сколько раз вызывался тот или иной метод с разной вариацией проверок
```ruby
RSpec.describe 'a random double' do
  it 'expects call stuntman.fall_off_ladder exactly 3 times' do
    stuntman = double("Mr. Danger")
    allow(stuntman).to receive_messages(fall_off_ladder: 'Ouch', light_on_fire: true)
    
    expect(stuntman).to receive(:fall_off_ladder).exactly(3).times

    stuntman.fall_off_ladder
    stuntman.fall_off_ladder
    stuntman.fall_off_ladder
  end

  it 'expects call stuntman.fall_off_ladder at least 2 times' do
    stuntman = double("Mr. Danger")
    allow(stuntman).to receive_messages(fall_off_ladder: 'Ouch', light_on_fire: true)

    expect(stuntman).to receive(:fall_off_ladder).at_least(2).times

    stuntman.fall_off_ladder
    stuntman.fall_off_ladder
    stuntman.fall_off_ladder
    stuntman.fall_off_ladder
  end
end
```
В первом тесте мы проверяем, что метод `fall_off_ladder` вызывался именно 3 раза, во втором тесте проверяем что этот метод вызывался минимум 2 раза.  
Важно заметить, что `receive` проверяет что метод вызывался в рамках текущего теста, т.е. не к моменту где стоит `expect(stuntman).to receive(:fall_off_ladder)`, а до конца текущего теста. Как проверяет это прямо на строке с expect будет описано далее.

## allow
В предыдущем примере было видно, что метод `allow` позволяет создавать заглушки на даблах, но он может также и переопределять функционал на любых объектах
```ruby
RSpec.describe 'allow method review' do
  it 'can customize return value for methods on doubles' do
    calculator = double
    allow(calculator).to receive(:add).and_return(15)

    expect(calculator.add).to eq(15)
    expect(calculator.add(3)).to eq(15)
    expect(calculator.add(-2, -3 -5)).to eq(15)
    expect(calculator.add('hello')).to eq(15)
  end

  it 'can stub one or more methods on a real object' do
    arr = [1, 2, 3]
    allow(arr).to receive(:sum).and_return(10)
    expect(arr.sum).to eq(10)

    arr.push(4)
    expect(arr).to eq([1, 2, 3, 4])
  end

  it 'can return multiple return values in sequence' do
    mock_array = double
    allow(mock_array).to receive(:pop).and_return(:a, :b, :c)
    expect(mock_array.pop).to eq(:a)
    expect(mock_array.pop).to eq(:b)
    expect(mock_array.pop).to eq(:c)
    expect(mock_array.pop).to eq(:c)
    expect(mock_array.pop).to eq(:c)
    expect(mock_array.pop).to eq(:c)
  end
end
```
В первом тесте мы задаем, что двойник `calculator` имеет метод `add` и в результате всегда возвращает 15 независимо от входных параметров.

Во втором тесте пример того, как `allow` может переопределять методы любых объектов. В данном случае мы переопределили метод sum у класса Array в рамках этого теста и он всегда возвращает 10 независимо от своего содержания.

В третье тесте чуть менее интуитивный понятный, но полезный функционал. Когда мы перечисляем параметры метода `and_return`, мы указываем порядок их возвращения при вызовах. Если мы указали параметры `:a, :b, :c`, то при первом вызове метода `mock_array.pop` мы получим `:a`, при втором `:b` и при третьем `:c`. Важный момент, что все последующие вызовы мы также будем получать последний указанный элемент, в нашем случае `:c`

## matching arguments
При установке заглушек можно также и указывать необходимые параметры
```ruby
RSpec.describe 'matching arguments' do
  it 'can return different values depending on the argument' do
    three_element_array = double # [1, 2, 3]

    allow(three_element_array).to receive(:first).with(no_args).and_return(1)
    allow(three_element_array).to receive(:first).with(1).and_return([1])
    allow(three_element_array).to receive(:first).with(2).and_return([1, 2])
    allow(three_element_array).to receive(:first).with(be >= 3).and_return([1, 2, 3])

    expect(three_element_array.first).to eq(1)
    expect(three_element_array.first(1)).to eq([1])
    expect(three_element_array.first(2)).to eq([1, 2])
    expect(three_element_array.first(3)).to eq([1, 2, 3])
    expect(three_element_array.first(100)).to eq([1, 2, 3])
  end
end
```
На 5 строке мы задаем, что если вызывается метод `first` без параметров, он возвращает `1`. На 6 и 7 строке мы указываем какой результат должен вернуть этот метод при разных входящих параметрах. На 8й задается результат, если входной параметр был больше 3х. 
## instance double
Более строгая версия double, которая на вход берет класс. Разница ее в том, что у этого дабла можно ставить заглушки только на те методы, что определены в переданном классе. Грубо говоря это возможность пользоваться объектом класса, не создавая реально этот объект.
```ruby
class Person
  def a(seconds)
    sleep(seconds)
    "Hello"
  end
end

RSpec.describe Person do
  describe 'regular double' do
    it 'can implement any method' do
      person = double(a: "Hello", b: 20)
      expect(person.a).to eq("Hello")
    end
  end

  describe 'instance double' do
    it 'can only implement methods that are defined on the class' do
      person = instance_double(Person)
      allow(person).to receive(:a).with(3).and_return("Hello")
      expect(person.a(3)).to eq("Hello")
    end
  end
end
```
В первом тесте для примера видно, что двойнику можно задать реализацию любого метода.
Во втором тесте используется `instance_double`, у которого можно переопределять только те методы, что указаны в классе `Person`.
Этот функционал позволяет более строго использовать двойника, исключая возможность в тесте задать ему такой функционал, который в реальности встречаться не будет никогда.

## class double
Похожая ситуация как и с instance double, только теперь мы строим заглушку класса
```ruby
class Deck
  def self.build
    # Business logic to build a whole bunch of cards
  end
end

class CardGame
  attr_reader :cards

  def start
    @cards = Deck.build
  end
end

RSpec.describe CardGame do
  it 'can only implement class methods that are defined on a class' do
    deck_klass = class_double(Deck, build: ['Ace', 'Queen']).as_stubbed_const

    expect(deck_klass).to receive(:build)
    subject.start
    expect(subject.cards).to eq(['Ace', 'Queen'])
  end
end
```
На 17й строке создается двойник класса `Deck`, также задается что возвращает метод `build`. Двойник класса также может подменять только методы, который указаны в оригинальном классе.

Другая важная и очень полезная особенность это `.as_stubbed_const`. Если задать двойник без нее, то в переменной `deck_klass` будет храниться двойник класса `Deck`, но оригинальный класс `Deck` также остался в памяти. В нашем случае внутри класса `CardGame` идет обращение к классу `Deck` и мы никак не можем на это повлиять. Но если у двойника вызвать метод `.as_stubbed_const`, то он собой заменяет оригинальный класс и когда `CardGame` внутри обращается к `Deck`, то попадет на нашего двойника.


## spy
Spy(шпион) это аналог double, который работает слегка иначе. В двойнике мы говорим ему какие методы он реализует и проверяем обращались ли к нему по его методам через `receive`. Шпиону не нужно указывать какие методы он реализует, он успешно сможет получать все методы, которые к нему обращаются, возвращая  при этом nil.
```ruby
RSpec.describe 'spies' do
  let(:animal) { spy('animal') }

  it 'confirms that a message has been received' do
    animal.eat_food
    expect(animal).to have_received(:eat_food)
    expect(animal).not_to have_received(:eat_human)
  end

  it 'resets between examples' do
    expect(animal).not_to have_received(:eat_food)
  end

  it 'retains the same functionality of a regular double' do
    animal.eat_food
    animal.eat_food
    animal.eat_food('Sushi')
    expect(animal).to have_received(:eat_food).exactly(3).times
    expect(animal).to have_received(:eat_food).at_least(2).times
    expect(animal).to have_received(:eat_food).with('Sushi')
    expect(animal).to have_received(:eat_food).once.with('Sushi')
  end
end
```
Еще одной отличительной чертой является использование метода `have_received` вместо `receive`. `have_received` проверяет был ли вызыван указанный метод на шпионе в данный момент, тогда как `receive` будет проверять, был ли вызыван этот метод до конца текущего теста.