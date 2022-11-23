# Библиотека гибкого парсинга дат по шаблону и форматированного вывода

## ЦЕЛЬ
Необходимо реализовать библиотеку, которая бы предоставляла API для парсинга и форматирования дат. Важно, чтобы библиотека
была расширяемой и поддерживала бы задание шаблонов для парсинга и форматирования. Говоря простым языком - добавить
любой нестандартный формат должно быть очень просто. Библиотека должна поддерживать не только даты, но и время, а также 
удобное API для работы с часовыми поясами.

### Пример (ваш API может отличаться)

```js
DateHelper.addTemplate('Example1', '${YYYY}.${MM}.${dd}');
DateHelper.addTemplate('Example2', '${YYYY}.${dd}.${MM}');

const d = DateHelper.parse('1989.10.18') // Date
DateHelper.fromat(d, 'Example2') // '1989.18.10'
```

## Решение 

```js
const d = new DateHelper();

d.addTemplate('Example1', 'YYYYMMDD');
d.addTemplate('Example2', 'DD.MM.YY HH:MM ');

d.format('Example1');
d.format('DD.MM.YYYY - hh:mm');
d.format('DD.MM.YYYY - hh:mm', new Date());

d.parse('1989.10.18');

d.format('Example2');

d.format('DD.MM.YYYY - hh:mm', d.parse('1980.10.18'));
```


#### P.S. Это лучше, чем ничего)
#### P.P.S. В планах переписать регулярки, и доработать библиотеку) 
###### P.P.P.S. Тут много заимствований)


### Код
```js
class DateHelper {
    constructor(date) {
        this.REGEX_PARSE = /^(\d{4})[-/]?(\d{1,2})?[-/]?(\d{0,2})[Tt\s]*(\d{1,2})?:?(\d{1,2})?:?(\d{1,2})?[.:]?(\d+)?$/
        this.REGEX_FORMAT = /\[([^\]]+)]|Y{1,4}|M{1,4}|D{1,2}|d{1,4}|H{1,2}|h{1,2}|a|A|m{1,2}|s{1,2}|Z{1,2}|SSS/g;
        this.templates = {
            default: 'DD.MM.YYYY',
            calendar: 'YYYY-MM-DD',
        };
        this.parse(date);
    }

    addTemplate(name, template = null) {
        if (!name) return "Invalid name";
        if (!template) return "Invalid template";
        this.templates[name] = template;
    }

    parse(date) {
        this.date = this.parseDate(date);
        this.init();
        return this.date;
    }

    parseDate(date) {
        if (date === null) return new Date(NaN);
        if (date === undefined) return new Date();
        if (date instanceof Date) return new Date(date);
        if (typeof date === 'string') {
          const d = date.match(this.REGEX_PARSE);
          if (d) {
            const m = d[2] - 1 || 0;
            const ms = (d[7] || '0').substring(0, 3);
            return new Date(d[1], m, d[3] || 1, d[4] || 0, d[5] || 0, d[6] || 0, ms);
          }
        }
        return new Date(date)
    }

    init() {
        this.year = this.date.getFullYear();
        this.month = this.date.getMonth();
        this.day = this.date.getDate();
        this.week = this.date.getDay();
        this.hour = this.date.getHours();
        this.minute = this.date.getMinutes();
        this.second = this.date.getSeconds();
        this.millisecond = this.date.getMilliseconds();
    }

    format(formatStr,d = null) {
        if (d) this.parse(d);
        if (this.date.toString() === 'Invalid Date') return 'Invalid Date';
        const str = this.templates[formatStr] || formatStr;
        const padStart = (string, length, pad) => {
            const s = String(string);
            if (!s || s.length >= length) return string;
            return `${Array((length + 1) - s.length).join(pad)}${string}`;
        };
        const getHour = num => (
            padStart(this.hour % 12 || 12, num, '0')
        );

        const matches = {
            YY: String(this.year).slice(-2),
            YYYY: this.year,
            M: this.month + 1,
            MM: padStart(this.month + 1, 2, '0'),
            D: this.day,
            DD: padStart(this.day, 2, '0'),
            d: String(this.week),
            H: String(this.hour),
            HH: padStart(this.hour, 2, '0'),
            h: getHour(1),
            hh: getHour(2),
            m: String(this.minute),
            mm: padStart(this.minute, 2, '0'),
            s: String(this.second),
            ss: padStart(this.second, 2, '0'),
            SSS: padStart(this.millisecond, 3, '0'),
        }

      return str.replace(this.REGEX_FORMAT, (match, $1) => $1 || matches[match]);
    }

    toString() {
      return this.date.toString();
    }
}
```

