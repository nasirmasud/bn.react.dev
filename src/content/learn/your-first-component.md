---
title: আপনার প্রথম কম্পোনেন্ট
---

<Intro>

*কম্পোনেন্ট* হলো React-এর মূল ধারণাগুলোর একটি। এগুলো ব্যবহারকারী ইন্টারফেস (UI) তৈরির ভিত্তি, যা আপনার React যাত্রা শুরু করার জন্য একটি চমৎকার স্থান!

</Intro>

<YouWillLearn>

- একটি কম্পোনেন্ট কী
- React অ্যাপ্লিকেশনে কম্পোনেন্ট কী ভূমিকা পালন করে
- কীভাবে আপনার প্রথম React কম্পোনেন্ট লিখবেন

</YouWillLearn>

## কম্পোনেন্ট: UI বিল্ডিং ব্লক {/*components-ui-building-blocks*/}

ওয়েব-এ, HTML আমাদেরকে বিল্ট-ইন ট্যাগগুলোর মাধ্যমে সমৃদ্ধ কাঠামোবদ্ধ ডকুমেন্ট তৈরি করতে দেয়, যেমন `<h1>` এবং `<li>`:

```html
<article>
  <h1>My First Component</h1>
  <ol>
    <li>Components: UI Building Blocks</li>
    <li>Defining a Component</li>
    <li>Using a Component</li>
  </ol>
</article>
```

এই মার্কআপটি এই প্রবন্ধটি `<article>`, এর শিরোনাম `<h1>`, এবং একটি (সংক্ষেপিত) বিষয়সূচী একটি ক্রম তালিকা `<ol>` হিসেবে উপস্থাপন করে। এই মার্কআপ, CSS-এর স্টাইল এবং JavaScript-এর ইন্টার‌্যাক্টিভিটি মিলিয়ে প্রতিটি UI উপাদানের পেছনে কাজ করে যা আপনি ওয়েবে দেখতে পান।
React আপনাকে আপনার মার্কআপ, CSS, এবং JavaScript কাস্টম "কম্পোনেন্ট" হিসেবে একত্রিত করতে দেয়, **আপনার অ্যাপের জন্য পুনঃব্যবহারযোগ্য UI উপাদান।** উপরের বিষয়সূচীর কোডটি একটি `<TableOfContents />` কম্পোনেন্টে পরিণত করা যেতে পারে যা আপনি প্রতিটি পৃষ্ঠায় রেন্ডার করতে পারেন। ভিতরে, এটি এখনও একই HTML ট্যাগগুলি ব্যবহার করে যেমন `<article>`, `<h1>`, ইত্যাদি।

HTML ট্যাগগুলির মতোই, আপনি পুরো পৃষ্ঠাগুলি ডিজাইন করতে কম্পোনেন্টগুলোকে কম্পোজ, অর্ডার এবং নেস্ট করতে পারেন। উদাহরণস্বরূপ, আপনি যে ডকুমেন্টেশন পৃষ্ঠাটি পড়ছেন তা React কম্পোনেন্ট দিয়ে তৈরি:

```js
<PageLayout>
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">Docs</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>
```

যখন আপনার প্রকল্প বড় হয়, আপনি দেখবেন যে আপনার অনেক ডিজাইন আপনি ইতিমধ্যে লিখিত কম্পোনেন্ট পুনঃব্যবহার করে তৈরি করতে পারবেন, যা আপনার ডেভেলপমেন্টকে দ্রুততর করবে। আমাদের উপরের বিষয়সূচীটি `<TableOfContents />` ব্যবহার করে যে কোনো স্ক্রিনে যুক্ত করা যেতে পারে! এমনকি React ওপেন সোর্স কমিউনিটির হাজারো কম্পোনেন্ট যেমন [Chakra UI](https://chakra-ui.com/) এবং [Material UI.](https://material-ui.com/) দিয়ে আপনার প্রকল্পটি দ্রুত শুরু করতে পারেন।

## কম্পোনেন্টকে সংজ্ঞায়িত করা {/*defining-a-component*/}

প্রথাগতভাবে, ওয়েব পেজ তৈরি করার সময়, ওয়েব ডেভেলপাররা তাদের কনটেন্টকে মার্কআপ করেন এবং তারপর কিছু জাভাস্ক্রিপ্ট যোগ করে ইন্টারঅ্যাকশন যুক্ত করেন। যখন ইন্টারঅ্যাকশন ওয়েবের জন্য একটি অতিরিক্ত সুবিধা ছিল, তখন এটি খুব ভাল কাজ করেছিল। তবে এখন অনেক সাইট এবং সমস্ত অ্যাপের জন্য এটি প্রত্যাশিত। React ইন্টারঅ্যাকশনকে প্রথম স্থানে রাখে, তবুও একই প্রযুক্তি ব্যবহার করে: **একটি React কম্পোনেন্ট একটি জাভাস্ক্রিপ্ট ফাংশন যা আপনি মার্কআপের সাথে _ছিটিয়ে_ দিতে পারেন।** নিচের উদাহরণটি দেখুন (আপনি নিচের উদাহরণটি সম্পাদনা করতে পারেন):

<Sandpack>

```js
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

```css
img { height: 200px; }
```

</Sandpack>

Here’s the translation of the provided text into Bengali:

এবং এখানে একটি কম্পোনেন্ট তৈরির পদক্ষেপ রয়েছে:

### ধাপ ১: কম্পোনেন্টটি এক্সপোর্ট করুন {/*step-1-export-the-component*/}

`export default` প্রিফিক্সটি একটি [মানক জাভাস্ক্রিপ্ট সিনট্যাক্স](https://developer.mozilla.org/docs/web/javascript/reference/statements/export) (যা React-এর জন্য বিশেষ নয়)। এটি আপনাকে একটি ফাইলে প্রধান ফাংশন চিহ্নিত করতে দেয় যাতে আপনি পরে এটি অন্য ফাইল থেকে আমদানি করতে পারেন। (কম্পোনেন্ট আমদানির বিষয়ে আরও জানুন [কম্পোনেন্ট আমদানি এবং রপ্তানি](/learn/importing-and-exporting-components)!)


### ধাপ ২: ফাংশনটি সংজ্ঞায়িত করুন {/*step-2-define-the-function*/}

`function Profile() { }` দিয়ে আপনি `Profile` নামে একটি জাভাস্ক্রিপ্ট ফাংশন সংজ্ঞায়িত করেন।

If you need any more translations or further assistance, just let me know!

<pitfall>

React কম্পোনেন্টগুলো সাধারণ JavaScript ফাংশন, তবে **এগুলোর নাম অবশ্যই বড় অক্ষরে শুরু করতে হবে** না হলে এগুলো কাজ করবে না!

</pitfall>

### Step 3: মার্কআপ যোগ করা {/*step-3-add-markup*/}

কম্পোনেন্টটি একটি `<img />` ট্যাগ রিটার্ন করে যার `src` এবং `alt` অ্যাট্রিবিউট রয়েছে। `<img />` এইচটিএমএল-এর মতো লেখা হয়, কিন্তু এটি আসলে পিছনে জাভাস্ক্রিপ্ট! এই সিনট্যাক্সকে [JSX](/learn/writing-markup-with-jsx) বলা হয়, এবং এটি আপনাকে জাভাস্ক্রিপ্টের মধ্যে মার্কআপ এম্বেড করতে দেয়।

রিটার্ন স্টেটমেন্টগুলি এই কম্পোনেন্টের মতো একটি লাইনে লেখা যেতে পারে:

```js
return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
```

But if your markup isn't all on the same line as the `return` keyword, you must wrap it in a pair of parentheses:

```js
return (
  <div>
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  </div>
);
```

<Pitfall>

Without parentheses, any code on the lines after `return` [will be ignored](https://stackoverflow.com/questions/2846283/what-are-the-rules-for-javascripts-automatic-semicolon-insertion-asi)!

</Pitfall>

## কম্পোনেন্ট এর ব্যবহার {/*using-a-component*/}

আপনি যখন `Profile` কম্পোনেন্টটি সংজ্ঞায়িত করেছেন, আপনি এটি অন্যান্য কম্পোনেন্টের ভিতরে নেস্ট করতে পারেন। উদাহরণস্বরূপ, আপনি একটি `Gallery` কম্পোনেন্ট এক্সপোর্ট করতে পারেন যা একাধিক `Profile` কম্পোনেন্ট ব্যবহার করে:

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

### ব্রাউজার কী দেখে {/*what-the-browser-sees*/}

লেটার কেসিং-এর পার্থক্য লক্ষ্য করুন:
- `<section>` ছোট হাতের অক্ষরে, তাই React জানে আমরা একটি HTML ট্যাগ উল্লেখ করছি।
- `<Profile />` বড় হাতের `P` দিয়ে শুরু হয়েছে, তাই React জানে যে আমরা `Profile` নামের কম্পোনেন্টটি ব্যবহার করতে চাই।
এবং `Profile`-এ আরও HTML রয়েছে: `<img />`। শেষ পর্যন্ত, ব্রাউজার এটিই দেখে:

```html
<section>
  <h1>Amazing scientists</h1>
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
</section>
```

### কম্পোনেন্ট নেস্টিং এবং সংগঠিত করা {/*nesting-and-organizing-components*/}

কম্পোনেন্টগুলি নিয়মিত জাভাস্ক্রিপ্ট ফাংশন, তাই আপনি একি ফাইলে একাধিক কম্পোনেন্ট রাখতে পারেন। যখন কম্পোনেন্টগুলি তুলনামূলকভাবে ছোট বা একে অপরের সাথে ঘনিষ্ঠভাবে সম্পর্কিত হয় তখন এটি সুবিধাজনক। যদি এই ফাইলটি ব্যস্ত হয়ে যায়, তবে আপনি সর্বদা `Profile`-কে একটি পৃথক ফাইলে স্থানান্তরিত করতে পারেন। আপনি শীঘ্রই [ইম্পোর্ট পেজে](/learn/importing-and-exporting-components) এটি করতে শিখবেন।

যেহেতু `Profile` কম্পোনেন্টগুলি `Gallery` এর ভিতরে রেন্ডার হয়—এমনকি কয়েকবার!—আমরা বলতে পারি যে `Gallery` একটি **প্যারেন্ট কম্পোনেন্ট,** যা প্রতিটি `Profile` কে "চাইল্ড" হিসেবে রেন্ডার করছে। এটি React-এর একটি অংশ: আপনি একবার একটি কম্পোনেন্ট সংজ্ঞায়িত করতে পারেন, এবং তারপর এটি যতবার এবং যত স্থানে ইচ্ছা ব্যবহার করতে পারেন।

<Pitfall>

কম্পোনেন্টগুলো অন্যান্য কম্পোনেন্ট রেন্ডার করতে পারে, তবে **আপনি কখনোই তাদের সংজ্ঞাগুলো নেস্ট করবেন না:**

```js {2-5}
export default function Gallery() {
  // 🔴 Never define a component inside another component!
  function Profile() {
    // ...
  }
  // ...
}
```

উপরের খন্ড-কোডটি [অত্যন্ত ধীর এবং বাগ সৃষ্টি করে।](/learn/preserving-and-resetting-state#different-components-at-the-same-position-reset-state) পরিবর্তে, প্রতিটি কম্পোনেন্ট শীর্ষ স্তরে সংজ্ঞায়িত করুন:

```js {5-8}
export default function Gallery() {
  // ...
}

// ✅ Declare components at the top level
function Profile() {
  // ...
}
```

যখন একটি চাইল্ড কম্পোনেন্টের একটি প্যারেন্ট থেকে কিছু ডেটার প্রয়োজন হয়, [এটি প্রপসের মাধ্যমে পাঠান](/learn/passing-props-to-a-component) সংজ্ঞাগুলি নেস্ট করার পরিবর্তে।

</Pitfall>

<DeepDive>

#### কম্পোনেন্টগুলি পুরোপুরি। {/*components-all-the-way-down*/}

আপনার React অ্যাপ্লিকেশন একটি "রুট" কম্পোনেন্ট দিয়ে শুরু হয়। সাধারণত, এটি নতুন প্রকল্প শুরু করার সময় স্বয়ংক্রিয়ভাবে তৈরি হয়। উদাহরণস্বরূপ, আপনি যদি [CodeSandbox](https://codesandbox.io/) ব্যবহার করেন বা [Next.js](https://nextjs.org/) ফ্রেমওয়ার্ক ব্যবহার করেন, তবে রুট কম্পোনেন্টটি `pages/index.js`-এ সংজ্ঞায়িত হয়। এই উদাহরণগুলিতে, আপনি রুট কম্পোনেন্টগুলি এক্সপোর্ট করছেন।

বেশিরভাগ React অ্যাপ্লিকেশন পুরোপুরি কম্পোনেন্ট ব্যবহার করে। এর মানে হল আপনি শুধুমাত্র পুনরায় ব্যবহারযোগ্য অংশগুলির জন্য যেমন বোতামগুলির জন্য কম্পোনেন্ট ব্যবহার করবেন না, বরং বড় অংশগুলির জন্য যেমন সাইডবার, তালিকা এবং অবশেষে সম্পূর্ণ পৃষ্ঠা! কম্পোনেন্টগুলি UI কোড এবং মার্কআপ সংগঠিত করার একটি সুবিধাজনক উপায়, এমনকি যদি তাদের মধ্যে কিছু কেবল একবারই ব্যবহার করা হয়।

[React-ভিত্তিক ফ্রেমওয়ার্কগুলি](/learn/start-a-new-react-project) এটি একটি পদক্ষেপ এগিয়ে নিয়ে যায়। খালি HTML ফাইল ব্যবহার করার পরিবর্তে এবং React-কে JavaScript দিয়ে পৃষ্ঠাটি "নিয়ন্ত্রণ" করতে দেওয়ার পরিবর্তে, তারা *অটোমেটিক্যালি* আপনার React কম্পোনেন্টগুলি থেকে HTML তৈরি করে। এটি আপনার অ্যাপ্লিকেশনকে JavaScript কোড লোড হওয়ার আগে কিছু কনটেন্ট প্রদর্শন করার অনুমতি দেয়।

তবুও, অনেক ওয়েবসাইট শুধুমাত্র বিদ্যমান HTML পৃষ্ঠাগুলিতে [ইন্টারঅ্যাকটিভিটি যুক্ত করতে React ব্যবহার করে।](/learn/add-react-to-an-existing-project#using-react-for-a-part-of-your-existing-page) তাদের সম্পূর্ণ পৃষ্ঠার জন্য একটি একক রুট কম্পোনেন্টের পরিবর্তে অনেক রুট কম্পোনেন্ট রয়েছে। আপনি যতটুকু—অথবা যত কম—React ব্যবহার করতে চান ততটুকু ব্যবহার করতে পারেন।

</DeepDive>

<Recap>

আপনি মাত্রই React-এর প্রথম টেস্ট পেয়েছেন! কিছু মূল পয়েন্ট পুনরায় সংক্ষেপে দেখা যাক।
- React আপনাকে কম্পোনেন্ট তৈরি করতে দেয়, **আপনার অ্যাপের জন্য পুনঃব্যবহারযোগ্য UI উপাদান।**
- একটি React অ্যাপে, প্রতিটি UI উপাদান একটি কম্পোনেন্ট।    
- React কম্পোনেন্ট হলো সাধারণ JavaScript ফাংশন শুধুমাত্র:    
    1. তাদের নাম সর্বদা বড় হাতের অক্ষরে শুরু হয়।
    2. তারা তাদের মার্কআপ JSX-এ রিটার্ন করে।


</Recap>



<Challenges>

#### Export the component {/*export-the-component*/}

এই স্যান্ডবক্সটি কাজ করে না কারণ রুট কম্পোনেন্টটি এক্সপোর্ট করা হয়নি:

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/lICfvbD.jpg"
      alt="Aklilu Lemma"
    />
  );
}
```

```css
img { height: 181px; }
```

</Sandpack>

সমাধান দেখার আগে এটি নিজে ঠিক করার চেষ্টা করুন!

<Solution>

ফাংশন সংজ্ঞার আগে `export default` যোগ করুন এইভাবে:

<Sandpack>

```js
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/lICfvbD.jpg"
      alt="Aklilu Lemma"
    />
  );
}
```

```css
img { height: 181px; }
```

</Sandpack>

আপনি হয়তো ভাবছেন কেন শুধু `export` লেখা এই উদাহরণটি ঠিক করতে যথেষ্ট নয়। আপনি [Importing and Exporting Components](/learn/importing-and-exporting-components) `export` এবং `export default` এর মধ্যে পার্থক্য শিখতে পারেন।

</Solution>

#### Fix the return statement {/*fix-the-return-statement*/}

এই `return` স্টেটমেন্টের কিছু সমস্যা আছে। আপনি কি এটি ঠিক করতে পারেন?

<Hint>

আপনি এটি ঠিক করার চেষ্টা করার সময় "Unexpected token" ত্রুটি পেতে পারেন। সেই ক্ষেত্রে, পরীক্ষা করুন যে সেমিকোলনটি বন্ধ করা বন্ধনীর *পরে* রয়েছে। `return ( )` এর ভিতরে সেমিকোলন রেখে দেওয়া ত্রুটি সৃষ্টি করবে।

</Hint>


<Sandpack>

```js
export default function Profile() {
  return
    <img src="https://i.imgur.com/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />;
}
```

```css
img { height: 180px; }
```

</Sandpack>

<Solution>

আপনি এই কম্পোনেন্টটি ঠিক করতে `return` স্টেটমেন্টটিকে এক লাইনে স্থানান্তরিত করতে পারেন এইভাবে:

<Sandpack>

```js
export default function Profile() {
  return <img src="https://i.imgur.com/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />;
}
```

```css
img { height: 180px; }
```

</Sandpack>

অথবা `return` এর ঠিক পরে খোলা বন্ধনীতে ফেরত দেওয়া JSX মার্কআপটি মোড়ানো:

<Sandpack>

```js
export default function Profile() {
  return (
    <img 
      src="https://i.imgur.com/jA8hHMpm.jpg" 
      alt="Katsuko Saruhashi" 
    />
  );
}
```

```css
img { height: 180px; }
```

</Sandpack>

</Solution>

#### Spot the mistake {/*spot-the-mistake*/}

`Profile` কম্পোনেন্টটি কীভাবে ডিক্লেয়ার এবং ব্যবহার করা হয়েছে তাতে কিছু ভুল রয়েছে। আপনি কি ভুলটি খুঁজে পেতে পারবেন? (React কীভাবে কম্পোনেন্টগুলোকে সাধারণ HTML ট্যাগ থেকে আলাদা করে তা মনে রাখার চেষ্টা করুন!)

<Sandpack>

```js
function profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <profile />
      <profile />
      <profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

<Solution>

React কম্পোনেন্টের নামগুলি একটি বড় অক্ষর দিয়ে শুরু হতে হবে।

`function profile()` কে `function Profile()`-এ পরিবর্তন করুন, এবং তারপর প্রতিটি `<profile />`-কে `<Profile />`-এ পরিবর্তন করুন:

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; }
```

</Sandpack>

</Solution>

#### Your own component {/*your-own-component*/}

শূন্য থেকে একটি কম্পোনেন্ট লিখুন। আপনি এটি যেকোনো বৈধ নাম দিতে পারেন এবং যেকোনো মার্কআপ ফেরত দিতে পারেন। যদি আপনার ধারণার অভাব হয়, তবে আপনি একটি `Congratulations` কম্পোনেন্ট লিখতে পারেন যা `<h1>Good job!</h1>` দেখায়। এটি এক্সপোর্ট করতে ভুলবেন না!

<Sandpack>

```js
// Write your component below!

```

</Sandpack>

<Solution>

<Sandpack>

```js
export default function Congratulations() {
  return (
    <h1>Good job!</h1>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
