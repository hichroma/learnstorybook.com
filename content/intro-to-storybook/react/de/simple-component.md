---
title: 'Baue eine einfache Komponente'
tocTitle: 'Einfache Komponente'
description: 'Baue eine einfache Komponente in Isolation'
commit: 403f19a
---

Beim Bauen unserer UI werden wir nach der [Component-Driven Development](https://blog.hichroma.com/component-driven-development-ce1109d56c8e) (CDD) Methodik vorgehen. Das it ein Vorgehen, in dem UIs "buttom up" entwickelt werden. Man beginnt mit Komponenten und endet mit Screens. CDD hilft dabei, die Komplexität zu begrenzen, mit der man beim Bauen einer UI konfrontiert wird.

## Task

![Task-Komponente in drei Zuständen](/intro-to-storybook/task-states-learnstorybook.png)

`Task` ist die zentrale Komponente in unserer App. Jede Aufgabe wird ein wenig anders angezeigt, abhängig davon, in welchem Zustand sie sich befindet. Wir zeigen eine ausgewählte (oder nicht ausgewählte) Checkbox an, einige Informationen über die Aufgabe und einen "Pin"-Button, der uns erlaubt, Aufgaben in der Liste hoch oder runter zu bewegen. Daraus ergeben sich folgende Props:

- `title` – ein String, der die Aufgabe beschreibt
- `state` - In welcher Liste befindet sich die Aufgabe aktuell und ist sie abgeschlossen?

Beim Entwickeln der `Task`-Komponente, schreiben wir zunächst unsere Test-Zustände, die den oben skizzierten möglichen Aufgaben-Typen entsprechen. Anschließend verwenden wir Storybook, um die Komponente mit gemockted Daten isoliert zu entwickeln. Wärend wir entwickeln, prüfen wir die Komponente in jedem möglichen Zustand auf ihre visuelle Erscheinung.

Dieses Vorgehen ähnelt der [testgetriebenen Entwicklung](https://de.wikipedia.org/wiki/Testgetriebene_Entwicklung) (TDD). Wir nennen es “[Visual TDD](https://blog.hichroma.com/visual-test-driven-development-aec1c98bed87)”.

## Los geht's

Lass uns zunächst eine Komponente für die Aufgaben anlegen sowie die zugehörige Story-Datei: `src/components/Task.js` und `src/components/Task.stories.js`.

Zunächst starten wir mit dem Grundgerüst von `Task`, in dem wir einfach die benötigten Attribute und die zwei Aktionen mitnehmen, die auf einer Aufgabe ausgeführt werden können (um sie zwischen Listen hin und her zu bewegen):

```javascript
// src/components/Task.js

import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className="list-item">
      <input type="text" value={title} readOnly={true} />
    </div>
  );
}
```

Oben rendern wir ein einfaches Markup für `Task`, basierend auf der bestehenden HTML-Struktur in der Todos-App.

Unten bilden wir die drei Test-Zustände, die `Task` einnehmen kann, in einer Story-Datei ab:

```javascript
// src/components/Task.stories.js

import React from 'react';
import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';

import Task from './Task';

export const task = {
  id: '1',
  title: 'Test Task',
  state: 'TASK_INBOX',
  updatedAt: new Date(2018, 0, 1, 9, 0),
};

export const actions = {
  onPinTask: action('onPinTask'),
  onArchiveTask: action('onArchiveTask'),
};

storiesOf('Task', module)
  .add('default', () => <Task task={task} {...actions} />)
  .add('pinned', () => <Task task={{ ...task, state: 'TASK_PINNED' }} {...actions} />)
  .add('archived', () => <Task task={{ ...task, state: 'TASK_ARCHIVED' }} {...actions} />);
```

Es gibt zwei grundlegende Ebenen, in denen Komponenten in Storybook origanisiert werden können: Die Komponente selbst und ihr untergeordnete Stories. Stell dir eine Story als Ausprägung einer Komponente vor. Du kannst so viele Stories für eine Komponente anlegen, wie du brauchst.

- **Komponente**
  - Story
  - Story
  - Story

Um Storybook zu initiieren, rufen wir zunächst die Funktion `storiesOf()` auf, um die Komponente zu registrieren. Wir brauchen einen Anzeigenamen für die Komponente - der Name, der dann in der Seitenleiste der Storybook App angezeigt wird.

`action()` erlaubt uns, ein Callback zu erstellen, das im **Actions** Panel der Storybook UI erscheint, wenn man auf dieses klickt. Wenn wir also einen Pin-Button bauen, können wir so in der Test UI sehen, ob ein Button-Klick erfolgreich war.

Da wir allen Ausprägungen unserer Komponente immer das selbe Menge an Actions übergeben müssen, ist es naheliegend, sie in einer einzigen `actions`-Variable zusammenzufassen und die `{...actions}` Syntax von JSX ("spread attributes") zu verwenden, um alle Props auf einmal zu übergeben. `<Task {...actions}>` ist äquivalent zu `<Task onPinTask={actions.onPinTask} onArchiveTask={actions.onArchiveTask}>`.

Ein weiterer Vorteil, die `actions` einer Komponente in einer Variable zusammenzufassen ist, wie wir später sehen werden, dass man diese dann via `export` Stories anderer Komponenten zur Verfügung stellen kann, die diese Komponente wiederverwenden.

Unsere Stories definieren wir, indem wir `add()` einmal für jeden unserer Test-Zustände aufrufen, um damit eine Story zu generieren. Die Action Story ist eine Funktion, die ein gerendertes Element in einem definierten Zustand zurückgibt (z.B. eine Komponenten-Klasse mit einer Menge an Props) - genau wie eine [Stateless Functional Component](https://reactjs.org/docs/components-and-props.html) in React.

Beim Erstellen einer Story nutzen wir eine Basis-Aufgabe (`task`), um die Struktur der Aufgabe zu definieren, die unsere Komponente erwartet. Diese basiert üblicherweise auf realitätsnahen Beispiel-Daten. Noch einmal: Diese Struktur zu `export`-ieren erlaubt uns, sie später in weiteren Stories zu verwenden, wie wir noch sehen werden.

<div class="aside">
<a href="https://storybook.js.org/addons/introduction/#2-native-addons"><b>Actions</b></a> helfen dir, Interaktionen zu verifizieren, wenn du UI Komponenten isoliert entwickelst. Oftmals hast du bei der Entwicklung keinen Zugriff auf die Funktionen und den Zustand, die im Kontext deiner App existieren. Nutze <code>action()</code>, um sie als Stub zur Verfügung zu haben.
</div>

## Konfiguration

Wir müssen auch noch eine kleine Anpassung an der Storybook-Konfiguration (`.storybook/config.js`) vornehmen, so dass unsere `.stories.js` Dateien und unsere CSS Datei berücksichtigt werden. Standardmäßig sucht Storybook im Verzeichnis `/stories` nach Stories; Dieses Tutorial verwendet ein Namens-Schema äquivalent zum `.test.js` Namens-Schema, das von CRA für automatisierte Tests bevorzugt wird.

```javascript
// .storybook/config.js

import { configure } from '@storybook/react';
import '../src/index.css';

const req = require.context('../src', true, /\.stories.js$/);

function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

Sobald wir das erledigt und den Storybook Server neu gestartet haben, sollten die Testfälle für die drei Zustände von `Task` angezeigt werden:

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook//inprogress-task-states.mp4"
    type="video/mp4"
  />
</video>

## Die Zustände implementieren

Da wir Storybook jetzt eingerichtet, die Styles importiert und die Testfälle angelegt haben, können wir einfach damit loslegen, das HTML der Komponente zu implementieren, so dass es dem Design entspricht.

Die Komponente ist noch immer sehr einfach gehalten. Schreib zunächst den Code, um das Design zu erhalten, ohne dass wir zu sehr ins Detail gehen: 

```javascript
// src/components/Task.js

import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className={`list-item ${state}`}>
      <label className="checkbox">
        <input
          type="checkbox"
          defaultChecked={state === 'TASK_ARCHIVED'}
          disabled={true}
          name="checked"
        />
        <span className="checkbox-custom" onClick={() => onArchiveTask(id)} />
      </label>
      <div className="title">
        <input type="text" value={title} readOnly={true} placeholder="Input title" />
      </div>

      <div className="actions" onClick={event => event.stopPropagation()}>
        {state !== 'TASK_ARCHIVED' && (
          <a onClick={() => onPinTask(id)}>
            <span className={`icon-star`} />
          </a>
        )}
      </div>
    </div>
  );
}
```

Das obige zusätzliche Markup, zusammen mit dem CSS, das wir zuvor imporiert haben, resultiert in folgender Darstellung:

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/finished-task-states.mp4"
    type="video/mp4"
  />
</video>

## Datenstruktur spezifizieren

Es ist üblich, `propTypes` in React zu verwenden, um die Struktur der Daten zu spezifizieren, die eine Komponente erwartet. Das dient nicht nur als Dokumentation, sondern hilft auch dabei, Probleme früh abzufangen.

```javascript
// src/components/Task.js

import React from 'react';
import PropTypes from 'prop-types';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  // ...
}

Task.propTypes = {
  task: PropTypes.shape({
    id: PropTypes.string.isRequired,
    title: PropTypes.string.isRequired,
    state: PropTypes.string.isRequired,
  }),
  onArchiveTask: PropTypes.func,
  onPinTask: PropTypes.func,
};
```

Nun wird beim Entwickeln eine Warnung angezeigt, wenn die `Task` Komponente falsch verwendet wird.

<div class="aside">
Alternativ kann man hierfür auch JavaScript Typisierung verwenden, wie z.B. TypeScript, um den Props der Komponente Typen zuzuweisen.
</div>

## Komponente erstellt!

Jetzt haben wir erfolgreich eine Komponente gebaut, ohne dass wir einen Server oder unsere gesamte Frontend App dazu benötigt hätten. Als nächstes müssen wir die verbleibenden Taskbox Komponenten auf die gleiche Weise bauen, eine nach der anderen.

Wie du siehst, ist es recht schnell und einfach möglich, eine Komponente in Isolation zu bauen. Dadurch können wir UIs bauen, die schicker, qualitativ hochwertiger und weniger fehleranfällig sind, weil es möglich ist, in die Tiefe zu gehen und jeden möglichen Zustand abzutesten.

## Automatisiertes Testen

Storybook hat uns eine tolle Möglichkeit gegeben, unsere Anwendung visuell zu testen während wir sie entwickeln. Die 'Stories' werden und dabei helfen, sicherzustellen, dass die Darstellung unserer `Task` Komponente nicht zerschossen wird, während wir unsere App weiter entwickeln. Allerdings ist das im Moment noch ein vollständig manueller Vorgang und irgendjemand muss sich die Mühe machen, alle Testzustände durchzuklicken, um sicherzustellen, dass alles korrekt gerendert wird und keine Fehler oder Warnungen auftreten. Können wir das nicht automatisieren?

### Snapshot testing

Snapshot testing refers to the practice of recording the “known good” output of a component for a given input and then flagging the component whenever the output changes in future. This complements Storybook, because it’s a quick way to view the new version of a component and check out the changes.

<div class="aside">
Make sure your components render data that doesn't change, so that your snapshot tests won't fail each time. Watch out for things like dates or randomly generated values.
</div>

With the [Storyshots addon](https://github.com/storybooks/storybook/tree/master/addons/storyshots) a snapshot test is created for each of the stories. Use it by adding a development dependency on the package:

```bash
yarn add --dev @storybook/addon-storyshots react-test-renderer require-context.macro
```

Then create an `src/storybook.test.js` file with the following in it:

```javascript
// src/storybook.test.js

import initStoryshots from '@storybook/addon-storyshots';
initStoryshots();
```

You'll also need to use a [babel macro](https://github.com/kentcdodds/babel-plugin-macros) to ensure `require.context` (some webpack magic) runs in Jest (our test context). Install it with:

```bash
yarn add --dev babel-plugin-macros
```

And enable it by adding a `.babelrc` file in the root folder of your app (same level as `package.json`)

```json
// .babelrc

{
  "plugins": ["macros"]
}
```

Then update `.storybook/config.js` to have:

```js
// .storybook/config.js

import { configure } from '@storybook/react';
import requireContext from 'require-context.macro';

import '../src/index.css';

const req = requireContext('../src/components', true, /\.stories\.js$/);

function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

(Notice we've replaced `require.context` with a call to `requireContext` imported from the macro).

Once the above is done, we can run `yarn test` and see the following output:

![Task test runner](/intro-to-storybook/task-testrunner.png)

We now have a snapshot test for each of our `Task` stories. If we change the implementation of `Task`, we’ll be prompted to verify the changes.