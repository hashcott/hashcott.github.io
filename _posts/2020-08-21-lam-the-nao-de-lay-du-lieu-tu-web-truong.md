---
layout: post
title: How to scrape data from your university's website
subtitle: Section 1 of the series "What can I do with my university's website?"
tags: [code]
comments: true
---

# Getting ready

## Background knowledge

As with anything, before you start you need some basics (though after my general-ed classes I feel like a blank sheet of paper 😭😭😭).

1. ### **[What is Puppeteer? Can you eat it?](https://github.com/puppeteer/puppeteer/blob/main/README.md)**

   **I found this series pretty basic and easy to follow for newcomers (in Vietnamese):**

- [Getting started with Puppeteer](https://toidicodedao.com/2017/12/12/puppeteer-headless-chrome-api-phan-1/)
- [Practicing with Puppeteer](https://toidicodedao.com/2017/12/19/puppeteer-headless-chrome-api-phan-2-cao-du-lieu-kenh14/)

- **[Puppeteer](https://github.com/puppeteer/puppeteer/blob/main/README.md)** is a very powerful library, but it also has a huge number of APIs =.= Just looking at them makes your head spin 😰. Luckily the documentation is detailed and easy to read, so it's manageable 🤗🤗🤗. Unlike React (which we'll work with in the next section), where you'd be totally lost.

  - The Puppeteer APIs we care about:
    - **[puppeteer.launch](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#puppeteerlaunchoptions)**: Opens Chrome so we can start playing around. Returns a Browser object.
    - **[browser.newPage](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#browsernewpage)**: Opens a new tab in Chrome. Returns a Page object.
    - **[browser.close](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#browserclose)**: Closes the browser (saves you closing it by hand).
    - **[page.goto](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#pagegotourl-options)**: Navigates to a page. The waitUntil parameter matters: it decides whether we wait until the page has just loaded, or until all its JavaScript and images have loaded.
    - **[page.waitForSelector](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#pagewaitforselectorselector-options)**: Waits until a given selector has loaded.
    - **[page.evaluate](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#pageevaluatepagefunction-args)**: The most important API. It runs a script inside the browser and returns the result.
    - **[page.type](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#pagetypeselector-text-options)**: Types into an input. Useful for logging in, searching, etc.
    - **[page.click](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#pageclickselector-options)**: Clicks a button, like the login button.
    - **[page.close](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#pagecloseoptions)**: Closes the page.

2. ### **[Lodash](https://lodash.com/docs/4.17.15)**
   A library that makes working with arrays and objects simpler.
3. ### **[Moment](https://momentjs.com/docs/)**
   Makes working with dates and times simpler.
4. ### **[ava](https://www.npmjs.com/package/ava)**

   AVA is a test runner for Node.js.

   Example:

   ```
    const test = require('ava');
    test('basic', async t => {
        t.is(3+12, 15);
    });
   ```

## Your toolkit

Can't do without it!!! First, pick an editor. There are plenty; use whichever you're used to. And if you're a pro, just use Notepad :V

- Editor:
  - [Visual Studio Code](https://code.visualstudio.com/) (recommended)
  - [Atom](https://atom.io/)
  - [Sublime Text](https://www.sublimetext.com/)
  - and so on :v
- Runtime: [Node.js](https://nodejs.org/en/). Download and install the LTS version for stability. If you're on an old version that doesn't support [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), please update.
  - [Asynchronous JavaScript: from callbacks and promises to observables](https://www.youtube.com/watch?v=qW3raOCefms) (in Vietnamese). This video by Tiep Phan explains it in great detail.

# Let's go! Time to take the university website apart.

**The university website:** http://dkh.tlu.edu.vn/CMCSoft.IU.Web.Info/Home.aspx

Other universities whose websites run on CMC's software can follow the same steps with a few small tweaks 😜😜😜

## Setting up the project

1. **Run the commands below to create the project's root folder and initialize npm**

```
mkdir student-api
cd student-api
npm init -y
```

2.  **Install the libraries the project needs**

```
npm i --save puppeteer moment moment-timezone lodash ava
```

3. **The project structure will look like this**

- root
  - pages: the code that runs on each page
  - util: helper libraries
  - selectors: selectors for the elements we interact with
  - test: tests

4. **Create the project skeleton**

- Create _index.js_ in **root**:

```
// Require dependencies
const puppeteer = require("puppeteer");

// Require custom lib

// Main

class StudentAPI {
  // Constructor
  constructor(opts = {}) {
    this._opts = opts;
    this._user = null;
  }

  // Whether a user is logged in => boolean
  get isAuthenticated() {
    return !!this._user;
  }
  // Returns the user
  get User() {
    return this._user;
  }

  // Launch the browser
  async Browser() {
    // Reuse the existing browser if there is one, otherwise launch a new one
    if (!this._browser) {
      this._browser =
        this._opts.browser || (await puppeteer.launch(this._opts.puppeteer));
    }
    return this._browser;
  }

  // Close the browser when we're done
  async Close() {
    const browser = await this.Browser();
    await browser.close();

    this._browser = null;
    this._user = null;
  }
}
module.exports = StudentAPI;
```

Now add _index.js_ in the **test** folder:

```
const test = require("ava");
const StudentAPI = require("../");

test("basic", (t) => {
  const api = new StudentAPI();
  t.is(api.isAuthenticated, false);
  t.is(api.user, null);
  console.log(api.isAuthenticated, api.user);
});
```

Run `npm run test`. Well done, the result is exactly as expected:

```
1 tests passed
```

## Now let's dig into what the university website has

The logic: first we go to the login page, then to the timetable page, and scrape it. Sounds easy already, so let's get to it.

## _Handling login_

1. First, let's look at the structure of the login page.

![Login page](/img/student-api/login.png)

We can easily spot the selectors of the username and password **_inputs_** and the login **_button_**:

```
  #txtUserName
  #txtPassword
  #btnSubmit
```

Create _LoginSelector.js_ in the **selectors** folder:

```
module.exports = {
  account: "#txtUserName",
  password: "#txtPassword",
  btnLogin: "#btnSubmit",
  userRole: "#PageHeader1_lblUserRole"
};
```

Now we'll use Puppeteer's **[page.goto](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#pagegotourl-options)**, **[page.type](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#pagetypeselector-text-options)** and **[page.click](https://github.com/puppeteer/puppeteer/blob/v5.2.1/docs/api.md#pageclickselector-options)** to log in.

Create _Login.js_ in **pages**:

```
const {
  account,
  password,
  btnLogin,
  userRole,
} = require("../selectors/LoginSelector");

module.exports = async (browser, { idUser, passwordUser } = user) => {
  // Open a new page
  const page = await browser.newPage();
  // Go to the login page
  await page.goto("http://dkh.tlu.edu.vn/cmcsoft.iu.web.info/Login.aspx");
  // Fill in the login form
  await page.type(account, idUser);
  await page.type(password, passwordUser);
  // Click the login button
  await page.click(btnLogin);
  await page.waitForSelector(userRole);

  // Check whether the login succeeded
  const element = await page.$(userRole);
  const text = await (await element.getProperty("textContent")).jsonValue();
  if (!text.length) throw new Error("Login failed");
  await page.close();
};

```

Now add the method to **index.js**:

```
...

// Require custom lib
const LoginPage = require("./pages/Login");
// Main
class StudentAPI {
  ...
  // Log in
  async Login(user) {
    const browser = await this.Browser();
    // If no error is thrown (login succeeded), this._user is set
    try {
      await LoginPage(browser, user);
      this._user = user;
    } catch (error) {
      throw new Error(error.message);
    }
  }
  ...
}
module.exports = StudentAPI;

```

Now go to the **test** folder and add this to _index.js_. Remember to put in your own username and password:

```
....
test("authenicated", async (t) => {
  const api = new StudentAPI();
  const user = { idUser: "username", passwordUser: "password" };
  await api.Login(user);
  // If login succeeded, isAuthenticated returns true
  t.is(api.isAuthenticated, true);
  t.is(api.user, user);
});
```

The result :v Well done!!! Runs smoothly:

```
2 tests passed
```

Yeahhhhhhhhhhhhh!!! 🤟🤟🤟 With just a few lines of code we can log in automatically 😜😜😜. The browser we launched now keeps the cookies, so let's head to the timetable page and scrape it.

## _Handling the timetable_

Once we're on the page http://dkh.tlu.edu.vn/CMCSoft.IU.Web.Info/Reports/Form/StudentTimeTable.aspx

![Timetable DOM](/img/student-api/domtable.png)

We can see the DOM structure:

```
#gridRegistered tr
```

That means every **row** of the table is wrapped by **#gridRegistered**, so a single **querySelectorAll** gets us all the data. Then we convert that NodeList into an array.

![Raw data](/img/student-api/DLTho.png)

```
Each row (tr) has several table cells (td). Run querySelectorAll on them the same way to get the raw data, then process it.
```

Let's open the console and try fetching and processing the data:

```
 // Holds the timetable for every course
    let timeTable = [];

    // Regexes that extract the dates and class times
    const data_pattern = /(.+?) đến (.+?):/;
    const time_pattern = /Thứ ([0-9]) tiết ([0-9,]+?) \((.+?)\)/;
    const location_pattern = /\(([0-9])+\)\n(.+)/g;

    // Grab the timetable HTML for each course
    let timeTableHTML = [
      ...document.querySelectorAll("#gridRegistered tr"),
    ].slice(1, 11);

    for (let html of timeTableHTML) {
      let subjectHTML = [...html.querySelectorAll("td")];

      // Turn each course's cells into text
      let [
        STT,
        lopHocPhan,
        hocPhan,
        thoiGian,
        diaDiem,
        siSo,
        daDk,
        soTc,
        hocPhi,
        ghiChu,
      ] = subjectHTML.map((sj) => sj.innerText.trim());

      // Save each course's timetable
      timeTable.push({
        STT,
        lopHocPhan,
        hocPhan,
        thoiGian,
        diaDiem,
        siSo,
        daDk,
        soTc,
        hocPhi,
        ghiChu,
      });
    }

    /* Split each course's schedule into its study phases */
    timeTable.forEach((data, i) => {
      // Normalize the times
      let getTimePhase = data.thoiGian.split("Từ");
      getTimePhase = getTimePhase.filter((time) => time != "");
      let phases = [];
      for (let time of getTimePhase) {
        let timePhase = time.split("\n");
        timePhase = timePhase.filter((time) => time !== "");
        let ranges = [];
        let [, start, end] = data_pattern
          .exec(timePhase.shift())
          .map((x) => x.trim());
        do {
          let [, day, periods, type] = time_pattern.exec(timePhase.shift());
          periods = periods.split(",");
          ranges.push({ day, periods, type });
        } while (timePhase.length > 0);
        phases.push({ start, end, ranges, location: "" });
      }

      // Normalize the locations
      if (phases.length > 1) {
        let matches;
        do {
          let matches = location_pattern.exec(data.diaDiem);
          if (matches) {
            let [, phase, location] = matches;
            phases[+phase - 1].location = location;
          }
        } while (matches);
      } else {
        phases[0].location = data.diaDiem;
      }
      timeTable[i].thoiGian = phases;
      delete timeTable[i].diaDiem;
    });
```

Let's run it 😅 and hope we get what we want:

![Timetable](/img/student-api/lich-hoc.png)

**Each course's data looks like this** (field names follow the website's Vietnamese column headings: class section, course code, schedule, class size, enrolled, credits, tuition, notes):

```
{
  STT: "1",
  lopHocPhan: "Bóng chuyền 1-2-19 (60.N01)",
  hocPhan: "TDUC141",
  thoiGian: [
    {
      start: "13/04/2020",
      end: "07/06/2020",
      ranges: [{ day: "3", periods: ["7", "8", "9"], type: "LT" }],
      location: "Sân BC 1 GDTC",
    },
  ],
  siSo: "45",
  daDk: "55",
  soTc: "1",
  hocPhi: "305.000",
  ghiChu: "",
}
```

Well done, it works, no surprise :v. Now we'll use Puppeteer's **_evaluate_** API to run it just like in the console.

Note: this API returns a Promise.

Create _StudentTimeTable.js_ in pages:

```
const {
  termSelector,
  tableSelector,
} = require("../selectors/StudentTimeTable");

module.exports = async (browser, term) => {
  const page = await browser.newPage();
  await page.goto(
    "http://dkh.tlu.edu.vn/CMCSoft.IU.Web.Info/Reports/Form/StudentTimeTable.aspx"
  );
  await page.type(termSelector, term);
  await page.waitForSelector(tableSelector);
  const timeTable = await page.evaluate(() => {
    *** PASTE THE CODE YOU JUST RAN IN THE CONSOLE HERE ***
    return timeTable;
  });
  await page.close();
  return timeTable;
};

```

Add this method to the main class in _index.js_:

```
...
  async StudentTimeTable(term) {
    const browser = await this.Browser();
    if (!this._timeTable) {
      try {
        this._timeTable = await StudentTimeTable(browser, term);
      } catch (error) {
        throw new Error("Could not fetch the course data");
      }
    }
    return this._timeTable;
  }
...
```

**Back to the tests: add this to index.js and run it:**

```
test("getStudentTimeTable", async (t) => {
  const api = new StudentAPI();
  const user = { idUser: "username", passwordUser: "password" };
  await api.Login(user);
  const lich = await api.StudentTimeTable("2_2019_2020");
  t.is(api.isAuthenticated, true);
  t.is(api.user, user);
  t.is(!!lich.length, true);
});

```

The result:

```
3 tests passed
```

TADAAAAAA 🥳🥳🥳 We've got the course data. Now let's turn it into a timeline.

## This part is your **challenge**. My sample code is below; there are many ways to do it.

```
const moment = require("moment-timezone");
const _ = require("lodash");
const periodBoard = require("./periodBoard");
moment.tz.setDefault("Asia/Ho_Chi_Minh");
moment.locale("vi-VN");

const parseTime = (time) => moment(time, "DD/MM/YYYY");

const generateTimestamps = (start, end, day) => {
  let timeLine = [];
  // Set the start day
  start.weekday(day);
  // While the start day is not past the end day, add it to the timeline
  while (start.isSameOrBefore(end)) {
    timeLine.push(start.clone());
    // Move ahead one week and check again
    start.add(1, "week");
  }
  return timeLine;
};

// Turn the dates into each class's start and end times
const generateClasses = (timeRanges, startPeriod, endPeriod) => {
  return timeRanges.map((timeStamps) => ({
    start: timeStamps
      .clone()
      .hour(periodBoard[startPeriod].start.hour)
      .minute(periodBoard[startPeriod].start.minute),
    end: timeStamps
      .clone()
      .hour(periodBoard[endPeriod].end.hour)
      .minute(periodBoard[endPeriod].end.minute),
  }));
};

// Build the timeline
const generateTimeline = (subjects) => {
  let timelines = [];
  for (let subject of subjects) {
    for (let phase of subject.thoiGian) {
      for (let range of phase.ranges) {
        console.log(range);
        let timeline = generateClasses(
          generateTimestamps(
            parseTime(phase.start),
            parseTime(phase.end),
            parseInt(range.day) - 2
          ),
          range.periods[0],
          range.periods[range.periods.length - 1]
        );
        timeline.forEach((timestamp) => {
          let data = { timestamp, ...subject, ...range };
          delete data.thoiGian;
          delete data.STT;
          timelines.push(data);
        });
      }
    }
  }
  timelines.sort((first, last) => first.timestamp.start - last.timestamp.start);
  return timelines;
};
//
const generateTimelineByDay = (timelines) => {
  let days = _.groupBy(timelines, (timeline) => {
    return timeline.timestamp.start.clone().startOf("day");
  });
  return days;
};

module.exports = { generateTimeline, generateTimelineByDay };


```

With my approach, the result looks like this:

```
{
  "Thu Jul 30 2020 00:00:00 GMT+0700": [
  {
    timestamp: {
      start: Moment<2020-07-30T07:00:00+07:00>,
      end: Moment<2020-07-30T08:45:00+07:00>
    },
    lopHocPhan: 'Lập trình nâng cao-2-19 (60TH4)',
    hocPhan: 'CSE381',
    siSo: '74',
    daDk: '73',
    soTc: '3',
    hocPhi: '915.000',
    ghiChu: '',
    day: '5',
    periods: [ '1', '2' ],
    type: 'LT'
  }
  ],
  ...
}
```

## That wraps up **_Section 1: "How to scrape data from your university's website"_**. I hope you learned something new. If you spot any mistakes, let me know so I can do better in Section 2.

### **GitHub repo**: https://github.com/hashcott/student-api

### **npm package**: https://www.npmjs.com/package/student-api

You can also check out: https://github.com/t-rekttt/tinchi-api
