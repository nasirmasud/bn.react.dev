---
title: 'আপনার সম্ভবত Effect প্রয়োজন নাও হতে পারে'
---

<Intro>

এফেক্ট হল রিয়্যাক্ট প্যারাডাইম থেকে বেরিয়ে আসার একটি মাধ্যম। এগুলি আপনাকে রিয়্যাক্টের বাইরে গিয়ে কিছু বাহ্যিক সিস্টেম যেমন কোনো নন-রিয়্যাক্ট উইজেট, নেটওয়ার্ক বা ব্রাউজারের DOM-এর সাথে আপনার কম্পোনেন্টগুলোকে সিঙ্ক্রোনাইজ করতে দেয়। যদি কোনো বাহ্যিক সিস্টেমের প্রয়োজন না থাকে (যেমন, কিছু প্রপস বা স্টেট পরিবর্তিত হলে কম্পোনেন্টের স্টেট আপডেট করতে চান), তাহলে ইফেক্ট প্রয়োজন হবে না। অপ্রয়োজনীয় ইফেক্ট অপসারণ করলে আপনার কোড অনুসরণ করা সহজ হবে, দ্রুত চলবে এবং কম ত্রুটি প্রবণ হবে।

</Intro>

<YouWillLearn>

* কেন এবং কীভাবে আপনার কম্পোনেন্ট থেকে অপ্রয়োজনীয় ইফেক্ট থেকে অপসারণ করবেন
* ইফেক্ট ছাড়াই ভারি কম্পিউটেশন কীভাবে ক্যাশ করবেন
* ইফেক্ট ছাড়াই কীভাবে কম্পোনেন্টের স্টেট রিসেট এবং সামঞ্জস্য করবেন
* কীভাবে ইভেন্ট হ্যান্ডলারের মধ্যে লজিক ভাগ করবেন
* কোন লজিক ইভেন্ট হ্যান্ডলারে স্থানান্তর করা উচিত
* কীভাবে পরিবর্তন সম্পর্কে প্যারেন্ট কম্পোনেন্টকে জানাবেন

</YouWillLearn>

## অপ্রয়োজনীয় ইফেক্ট কীভাবে অপসারণ করবেন {/*how-to-remove-unnecessary-effects*/}

এ দুটি সাধারণ ক্ষেত্রে ইফেক্টের প্রয়োজন হয় না:

* **রেন্ডার করার জন্য ডেটা ট্রান্সফার করতে ইফেক্টের প্রয়োজন নেই।** ধরুন, আপনি একটি তালিকা ফিল্টার করতে চান প্রদর্শনের আগে। এখানে তালিকা পরিবর্তিত হলে ইফেক্ট লিখে একটি স্টেট ভেরিয়েবল আপডেট করার চিন্তা আসতে পারে। তবে এটি অকার্যকর। স্টেট আপডেট হলে, React প্রথমে কম্পোনেন্ট ফাংশনগুলোকে কল করে ঠিক করে কী স্ক্রিনে দেখানো হবে। এরপর React সেই পরিবর্তনগুলো DOM-এ ["কমিট"](/learn/render-and-commit) করে স্ক্রিন আপডেট করে। তারপর React ইফেক্টগুলো চালায়। যদি ইফেক্ট তৎক্ষণাৎ স্টেট আপডেট করে, তাহলে পুরো প্রক্রিয়া আবার শুরু হবে! এই অপ্রয়োজনীয় রেন্ডার পাস এড়াতে, সব ডেটা কম্পোনেন্টের শীর্ষ স্তরে রূপান্তর করুন। আপনার প্রপস বা স্টেট পরিবর্তিত হলে এই কোড স্বয়ংক্রিয়ভাবে পুনরায় চলবে।

* **ইউজার ইভেন্ট হ্যান্ডল করতে ইফেক্টের প্রয়োজন নেই।** উদাহরণস্বরূপ, আপনি চাইছেন যে ইউজার কোনো পণ্য কিনলে একটি `/api/buy` POST রিকোয়েস্ট পাঠানো হবে এবং একটি নোটিফিকেশন দেখানো হবে। Buy বোতামের ক্লিক ইভেন্ট হ্যান্ডলারে, আপনি জানেন কী ঘটেছে। ইফেক্ট চালানোর সময়, আপনি জানেন না *ঠিক কী* ইউজার করেছেন (যেমন, কোন বোতামটি ক্লিক হয়েছে)। এ কারণেই সাধারণত ইউজার ইভেন্টগুলো সংশ্লিষ্ট ইভেন্ট হ্যান্ডলারে হ্যান্ডল করা হয়।

আপনাকে বাহ্যিক সিস্টেমের সাথে [সিঙ্ক্রোনাইজ](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events) করতে ইফেক্টের প্রয়োজন। উদাহরণস্বরূপ, আপনি একটি ইফেক্ট লিখে একটি jQuery উইজেটকে React স্টেটের সাথে সিঙ্ক্রোনাইজ রাখতে পারেন। এছাড়াও, ইফেক্ট ব্যবহার করে ডেটা ফেচ করা যায়: উদাহরণস্বরূপ, বর্তমান সার্চ কুয়েরির সাথে সার্চ রেজাল্টকে সিঙ্ক্রোনাইজ করতে পারেন। মনে রাখবেন, আধুনিক [ফ্রেমওয়ার্কগুলো](/learn/start-a-new-react-project#production-grade-react-frameworks) সরাসরি কম্পোনেন্টে ইফেক্ট লেখার চেয়ে আরও এফিশিয়েন্ট বিল্ট-ইন ডেটা ফেচিং পদ্ধতি প্রদান করে।

সঠিক ধারণা গড়ে তুলতে, আসুন কিছু সাধারণ উদাহরণ দেখি!

### প্রপস বা স্টেটের উপর ভিত্তি করে স্টেট আপডেট করা {/*updating-state-based-on-props-or-state*/}

ধরুন, আপনার একটি কম্পোনেন্টে দুটি স্টেট ভেরিয়েবল রয়েছে: `firstName` এবং `lastName`। আপনি এগুলো যোগ করে একটি `fullName` তৈরি করতে চান। এর পাশাপাশি, `firstName` বা `lastName` পরিবর্তিত হলে `fullName` আপডেট হওয়া উচিত। আপনার প্রথম ধারণা হতে পারে একটি `fullName` স্টেট ভেরিয়েবল যোগ করা এবং একটি ইফেক্টে এটি আপডেট করা:

```js {5-9}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

এটি প্রয়োজনের তুলনায় বেশি জটিল এবং অকার্যকর: এটি `fullName` এর পুরনো মান দিয়ে একটি সম্পূর্ণ রেন্ডার পাস করে, তারপর আপডেট হওয়া মান নিয়ে পুনরায় রেন্ডার করে। স্টেট ভেরিয়েবল এবং ইফেক্টটি সরিয়ে দিন:

```js {4-5}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Good: calculated during rendering
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

**যখন কিছু বিদ্যমান প্রপস বা স্টেট থেকে ক্যাল্কুলেট করা যায়, তখন [স্টেটে এটি রাখবেন না.](/learn/choosing-the-state-structure#avoid-redundant-state) পরিবর্তে, এটি রেন্ডার করার সময় ক্যাল্কুলেট করুন।** এর ফলে আপনার কোড দ্রুততর হয় (আপনি অতিরিক্ত "ক্যাসকেডিং" আপডেট এড়িয়ে যান), সহজতর হয় (কিছু কোড অপসারণ করেন), এবং কম ত্রুটি প্রবণ হয় (ভিন্ন স্টেট ভেরিয়েবলের সাথে সিঙ্ক্রোনাইজেশন সমস্যা এড়ান)। যদি এই পদ্ধতি আপনার জন্য নতুন মনে হয়, তাহলে [Thinking in React](/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state) ব্যাখ্যা করে কী কী স্টেটে থাকা উচিত।

### ভারি ক্যাল্কুলেশনগুলি ক্যাশ করা {/*caching-expensive-calculations*/}

এই কম্পোনেন্টটি `todos` প্রপস থেকে পাওয়া টাস্কগুলোকে নিয়ে `visibleTodos` গণনা করে এবং সেগুলোকে `filter` প্রপস অনুযায়ী ফিল্টার করে। আপনি মনে করতে পারেন যে ফলাফলটি স্টেটে সংরক্ষণ করা উচিত এবং একটি ইফেক্ট থেকে এটি আপডেট করা উচিত:

```js {4-8}
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

পূর্ববর্তী উদাহরণের মতো, এটি অপ্রয়োজনীয় এবং অকার্যকর। প্রথমে, স্টেট এবং ইফেক্টটি অপসারণ করুন:

```js {3-4}
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ This is fine if getFilteredTodos() is not slow.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

সাধারণত, এই কোডটি ঠিক আছে! তবে হয়তো `getFilteredTodos()` ধীর গতির অথবা আপনার কাছে অনেক `todos` রয়েছে। এ ক্ষেত্রে, আপনি চান না যে কিছু অসম্পর্কিত স্টেট ভেরিয়েবল যেমন `newTodo` পরিবর্তিত হলে `getFilteredTodos()` পুনরায় গণনা করা হোক।

আপনি একটি ভারি ক্যাল্কুলেশন ক্যাশ (অথবা ["মেমোইজ"](https://en.wikipedia.org/wiki/Memoization)) করতে পারেন এটি [`useMemo`](/reference/react/useMemo) হুকের মধ্যে স্থাপন করে:

```js {5-8}
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

অথবা, এক লাইনে লেখা:

```js {5-6}
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ Does not re-run getFilteredTodos() unless todos or filter change
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
  // ...
}
```

**এটি React-কে জানায় যে আপনি চান না যে অভ্যন্তরীণ ফাংশনটি পুনরায় চলুক যতক্ষণ না `todos` অথবা `filter` পরিবর্তিত হয়।** React প্রথম রেন্ডারের সময় `getFilteredTodos()` এর রিটার্ন ভ্যালু মনে রাখবে। পরবর্তী রেন্ডারগুলিতে, এটি চেক করবে যে `todos` অথবা `filter` ভিন্ন কিনা। যদি সেগুলো আগের মতো একই থাকে, তবে `useMemo` সর্বশেষ ফলাফলটি ফিরিয়ে দেবে যা এটি সংরক্ষণ করেছে। কিন্তু যদি সেগুলো ভিন্ন হয়, তবে React আবার অভ্যন্তরীণ ফাংশনটিকে কল করবে (এবং এর ফলাফল সংরক্ষণ করবে)।

যে ফাংশনটি আপনি [`useMemo`](/reference/react/useMemo) এর মধ্যে রাখেন তা রেন্ডার করার সময় চলে, তাই এটি শুধুমাত্র [শুদ্ধ ক্যাল্কুলেশনের জন্য](/learn/keeping-components-pure) কাজ করে।

<DeepDive>

#### কীভাবে জানা যাবে যে ক্যাল্কুলেশনটি ভারি কিনা? {/*how-to-tell-if-a-calculation-is-expensive*/}

সাধারণভাবে, যতক্ষন না আপনি হাজার হাজার অবজেক্ট তৈরি করছেন বা লুপ করছেন, এটি সম্ভবত ভারি নয়। যদি আপনি আরও নিশ্চিত হতে চান, তাহলে একটি কনসোল লগ যোগ করতে পারেন কোডের একটি অংশে সময় পরিমাপ করতে:

```js {1,3}
console.time('filter array');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array');
```

আপনার পরিমাপ করা ইন্টারঅ্যাকশনটি সম্পাদন করুন(যেমন, ইনপুটে টাইপ করা)। এরপর কনসোলে `filter array: 0.15ms` এর মতো লগ দেখতে পাবেন। যদি সামগ্রিক লগ করা সময় একটি গুরুত্বপূর্ণ পরিমাণে (যেমন, `1ms` বা তার বেশি) হয়, তাহলে সেই ক্যাল্কুলেশটি মেমোইজ করা অর্থপূর্ণ হতে পারে। পরীক্ষা হিসেবে, আপনি তারপর ক্যাল্কুলেশটি `useMemo` এর মধ্যে আবৃত করতে পারেন এবং যাচাই করতে পারেন যে সেই ইন্টারঅ্যাকশনের জন্য মোট লগ করা সময় কমেছে কিনা:

```js
console.time('filter array');
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // Skipped if todos and filter haven't changed
}, [todos, filter]);
console.timeEnd('filter array');
```
`useMemo` *প্রথম* রেন্ডারটি দ্রুত করবে না। এটি শুধুমাত্র আপডেটে অপ্রয়োজনীয় কাজ এড়াতে সাহায্য করে।

মনে রাখবেন, আপনার মেশিন হয়তো আপনার ব্যবহারকারীদের তুলনায় দ্রুত, তাই পারফরম্যান্স পরীক্ষার জন্য কৃত্রিমভাবে ধীরগতিতে পরীক্ষা করা ভালো। উদাহরণস্বরূপ, Chrome এ একটি [CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) অপশন প্রদান করে।

এছাড়াও মনে রাখবেন যে ডেভেলপমেন্টে পারফরম্যান্স পরিমাপ করা সবচেয়ে সঠিক ফলাফল দেবে না। (যেমন, [Strict Mode](/reference/react/StrictMode) চালু থাকলে আপনি প্রতিটি কম্পোনেন্টের রেন্ডার একবারের পরিবর্তে দুইবার দেখবেন।) সবচেয়ে সঠিক টাইমিং পেতে আপনার অ্যাপটি প্রোডাকশনের জন্য তৈরি করুন এবং এটি ব্যবহারকারীদের মতো ডিভাইসে পরীক্ষা করুন।

</DeepDive>

### প্রপ পরিবর্তিত হলে সমস্ত স্টেট রিসেট করা {/*resetting-all-state-when-a-prop-changes*/}

এই `ProfilePage` কম্পোনেন্ট একটি `userId` প্রপ গ্রহণ করে। পেজটিতে একটি মন্তব্য ইনপুট রয়েছে, এবং আপনি এর মান সংরক্ষণের জন্য একটি `comment` স্টেট ভেরিয়েবল ব্যবহার করেন। একদিন, আপনি একটি সমস্যা লক্ষ্য করেন: যখন আপনি এক প্রোফাইল থেকে অন্য প্রোফাইলে যান, `comment` স্টেটটি রিসেট হয় না। ফলে, ভুল করে অন্য ব্যবহারকারীর প্রোফাইলে মন্তব্য পোস্ট করা সহজ হয়ে যায়। এই সমস্যাটি সমাধান করতে, আপনি চান যে `userId` পরিবর্তিত হলে `comment` স্টেট ভেরিয়েবলটি পরিষ্কার হয়ে যাক।

```js {4-7}
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

এটি অকার্যকর কারণ `ProfilePage` এবং এর চাইল্ডরা প্রথমে পুরনো মান দিয়ে রেন্ডার হবে, এবং তারপর আবার রেন্ডার হবে। এটি জটিলও কারণ আপনাকে `ProfilePage`-এর ভিতরে থাকা প্রতিটি কম্পোনেন্টের স্টেটেও একই কাজ করতে হবে। উদাহরণস্বরূপ, যদি মন্তব্যের UIটি নেস্টেড থাকে, তবে আপনাকে নেস্টেড মন্তব্যের স্টেটও পরিষ্কার করতে হবে।

এর পরিবর্তে, আপনি React-কে জানাতে পারেন যে প্রতিটি ব্যবহারকারীর প্রোফাইল ধারণাগতভাবে একটি _ভিন্ন_ প্রোফাইল, `key` প্রদান করে। আপনার কম্পোনেন্টটি দুটি ভাগে ভাগ করুন এবং বাইরের কম্পোনেন্ট থেকে ভিতরের কম্পোনেন্টে একটি `key` অ্যাট্রিবিউট পাস করুন:

```js {5,11-12}
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState('');
  // ...
}
```

সাধারণত, একই স্থানে একই কম্পোনেন্ট রেন্ডার হলে React স্টেট সংরক্ষণ করে। **`Profile` কম্পোনেন্টে `userId`-কে `key` হিসেবে পাস করলে, আপনি React-কে বলছেন যে বিভিন্ন `userId` সহ দুটি `Profile` কম্পোনেন্টকে আলাদা কম্পোনেন্ট হিসেবে বিবেচনা করতে, যেগুলোর স্টেট শেয়ার হওয়া উচিত নয়।** যখনই `key` (যেটি আপনি `userId` হিসেবে সেট করেছেন) পরিবর্তিত হয়, React `Profile` কম্পোনেন্ট এবং এর সব চাইল্ডের DOM পুনরায় তৈরি করবে এবং [স্টেট রিসেট করবে](/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key)। এখন প্রোফাইলের মধ্যে নেভিগেট করার সময় `comment` ফিল্ড স্বয়ংক্রিয়ভাবে সাফ হয়ে যাবে।

এই উদাহরণে, শুধুমাত্র বাইরের `ProfilePage` কম্পোনেন্টটি প্রোজেক্টের অন্যান্য ফাইলের জন্য এক্সপোর্ট করা হয়েছে এবং দৃশ্যমান। `ProfilePage`-কে রেন্ডার করা কম্পোনেন্টগুলোর `key` পাস করার দরকার নেই: তারা `userId`-কে একটি সাধারণ প্রপ হিসেবে পাস করে। `ProfilePage` এটিকে `key` হিসেবে ভিতরের `Profile` কম্পোনেন্টে পাস করে, যা একটি ইমপ্লিমেন্টেশন ডিটেইল।

### প্রপ পরিবর্তিত হলে কিছু স্টেট সামঞ্জস্য করা {/*adjusting-some-state-when-a-prop-changes*/}

কখনও কখনও, প্রপ পরিবর্তিত হলে আপনি স্টেটের একটি অংশ রিসেট বা সামঞ্জস্য করতে চাইতে পারেন, তবে পুরোটা নয়।

এই `List` কম্পোনেন্টটি একটি `items` এর তালিকা প্রপ হিসেবে গ্রহণ করে এবং নির্বাচিত আইটেমটি `selection` স্টেট ভেরিয়েবলে রাখে। আপনি চান যখনই `items` প্রপ একটি ভিন্ন অ্যারে গ্রহণ করবে, `selection` কে `null` এ রিসেট করতে:

```js {5-8}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

এটিও আদর্শ নয়। প্রতি বার `items` পরিবর্তিত হলে, প্রথমে `List` এবং এর চাইল্ড কম্পোনেন্টগুলো পুরনো `selection` মান নিয়ে রেন্ডার হবে। তারপর React DOM আপডেট করবে এবং Effects চালাবে। শেষে `setSelection(null)` কলটি `List` এবং এর চাইল্ড কম্পোনেন্টগুলোকে পুনরায় রেন্ডার করবে, যা পুরো প্রক্রিয়াটি আবার শুরু করবে।

Effect মুছে ফেলার মাধ্যমে শুরু করুন। এর পরিবর্তে, সরাসরি রেন্ডারিংয়ের সময় স্টেটটি সামঞ্জস্য করুন।

```js {5-11}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

[আগের রেন্ডার থেকে তথ্য সংরক্ষণ করা](/reference/react/useState#storing-information-from-previous-renders) এভাবে বোঝা কঠিন হতে পারে, কিন্তু এটি একই স্টেটকে একটি Effect-এ আপডেট করার চেয়ে ভালো। উপরোক্ত উদাহরণে, `setSelection` সরাসরি রেন্ডার চলাকালীন কল করা হয়েছে। React `return` স্টেটমেন্টের পরে `List`-কে *তাৎক্ষণিকভাবে* পুনরায় রেন্ডার করবে। React এখনও `List`-এর চাইল্ডদের রেন্ডার করেনি বা DOM আপডেট করেনি, তাই এটি `List`-এর চাইল্ডদের পুরনো `selection` মান রেন্ডার করা এড়াতে সাহায্য করে।

যখন আপনি একটি রেন্ডারের সময় কম্পোনেন্ট আপডেট করেন, React রিটার্ন করা JSX ফেলে দেয় এবং তাৎক্ষণিকভাবে পুনরায় রেন্ডার করার চেষ্টা করে। খুব ধীর গতির পুনরাবৃত্তি এড়াতে, React কেবল রেন্ডারের সময় *একই* কম্পোনেন্টের স্টেট আপডেট করার অনুমতি দেয়। যদি আপনি রেন্ডারের সময় অন্য কোনো কম্পোনেন্টের স্টেট আপডেট করেন, তাহলে একটি ত্রুটি দেখতে পাবেন। একটি শর্ত যেমন `items !== prevItems` লুপ এড়ানোর জন্য প্রয়োজনীয়। আপনি এভাবে স্টেট সামঞ্জস্য করতে পারেন, তবে DOM পরিবর্তন করা বা টাইমআউট সেট করার মতো যেকোনো সাইড ইফেক্ট ইভেন্ট হ্যান্ডলার বা Effect-এ থাকা উচিত [কম্পোনেন্টকে পিওর রাখার জন্য।](/learn/keeping-components-pure)

**যদিও এই প্যাটার্নটি একটি Effect-এর চেয়ে আরও কার্যকর, বেশিরভাগ কম্পোনেন্টের এটিও প্রয়োজন হয় না।** যেভাবেই করুন না কেন, প্রপ বা অন্য কোনো স্টেটের ভিত্তিতে স্টেট সামঞ্জস্য করা আপনার ডেটা ফ্লো বুঝতে এবং ডিবাগ করতে আরও কঠিন করে তোলে। সর্বদা পরীক্ষা করুন যে আপনি [একটি key দিয়ে সমস্ত স্টেট রিসেট করতে পারেন কি না](#resetting-all-state-when-a-prop-changes) বা [রেন্ডারিংয়ের সময় সবকিছু গণনা করতে পারেন কি না](#updating-state-based-on-props-or-state)। উদাহরণস্বরূপ, নির্বাচিত *item* সংরক্ষণ এবং রিসেট করার পরিবর্তে আপনি নির্বাচিত *item ID* সংরক্ষণ করতে পারেন:

```js {3-5}
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

এখন স্টেট "সামঞ্জস্য" করার প্রয়োজন নেই। নির্বাচিত ID সহ আইটেমটি তালিকায় থাকলে, এটি নির্বাচিত থাকবে। যদি এটি না থাকে, তবে রেন্ডারিংয়ের সময় গণনা করা `selection` হবে `null` কারণ কোনো মেলানো আইটেম পাওয়া যায়নি। এই আচরণটি ভিন্ন, কিন্তু এটি যুক্তিসঙ্গতভাবে ভালো, কারণ `items` এর বেশিরভাগ পরিবর্তন সিলেকশকে সংরক্ষণ করে।

### ইভেন্ট হ্যান্ডলারগুলির মধ্যে লজিক শেয়ার করা {/*sharing-logic-between-event-handlers*/}

ধরি, আপনার একটি পণ্য পৃষ্ঠা আছে যার দুইটি বোতাম (Buy এবং Checkout) আছে, যেগুলো উভয়েই সেই পণ্যটি কেনার সুযোগ দেয়। আপনি চান যে ব্যবহারকারী পণ্যটি কার্টে রাখলেই একটি নোটিফিকেশন দেখানো হোক। উভয় বোতামের ক্লিক হ্যান্ডলারগুলিতে `showNotification()` কল করা পুনরাবৃত্তির মতো মনে হতে পারে, তাই আপনি এই লজিকটি একটি Effect-এ রাখতে লোভিত হতে পারেন:

```js {2-7}
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

এই Effect অপ্রয়োজনীয়। এটি সম্ভবত বাগও সৃষ্টি করবে। উদাহরণস্বরূপ, ধরুন আপনার অ্যাপটি পৃষ্ঠা রিফ্রেশের মধ্যে শপিং কার্টটি "মনে রাখে"। যদি আপনি একবার একটি পণ্য কার্টে যোগ করেন এবং পৃষ্ঠা রিফ্রেশ করেন, তবে নোটিফিকেশন আবারও দেখা যাবে। এই পণ্যটির পৃষ্ঠাটি রিফ্রেশ করার সময় এটি প্রতিবার দেখা যাবে। এর কারণ হল `product.isInCart` পৃষ্ঠা লোডের সময় ইতিমধ্যেই `true` থাকবে, তাই উপরে উল্লেখিত Effect `showNotification()` কল করবে।

**যখন আপনি নিশ্চিত না হন যে কোন কোডটি Effect-এ থাকা উচিত বা ইভেন্ট হ্যান্ডলারে, তখন নিজেকে জিজ্ঞাসা করুন *কেন* এই কোডটি চলতে হবে। শুধু সেই কোডের জন্য Effects ব্যবহার করুন যা *ব্যবহারকারীর জন্য* কম্পোনেন্টটি প্রদর্শিত হওয়ার কারণে চলতে হবে।** এই উদাহরণে, নোটিফিকেশনটি ব্যবহারকারী *বোতামটি চাপার* কারণে প্রদর্শিত হওয়া উচিত, পৃষ্ঠা প্রদর্শিত হওয়ার কারণে নয়! Effect টি মুছে ফেলুন এবং উভয় ইভেন্ট হ্যান্ডলারে কল করার জন্য একটি ফাংশনে শেয়ার করা লজিকটি রাখুন:

```js {2-6,9,13}
function ProductPage({ product, addToCart }) {
  // ✅ Good: Event-specific logic is called from event handlers
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

এটি অপ্রয়োজনীয় Effect টি মুছে ফেলে এবং বাগটিও ঠিক করে।

### পোস্ট রিকোয়েস্ট পাঠানো {/*sending-a-post-request*/}

এই `Form` কম্পোনেন্টটি দুটি ধরনের POST অনুরোধ পাঠায়। এটি মাউন্ট হওয়ার সময় একটি অ্যানালিটিক্স ইভেন্ট পাঠায়। ফর্মটি পূরণ করে যখন আপনি সাবমিট বোতামটি ক্লিক করেন, এটি `/api/register` এন্ডপয়েন্টে একটি POST অনুরোধ পাঠাবে:

```js {5-8,10-16}
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic should run because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 🔴 Avoid: Event-specific logic inside an Effect
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

আগের উদাহরণের মতো একই মানদণ্ড এখানে প্রয়োগ করা যাক।

অ্যানালিটিক্স POST অনুরোধটি Effect এ থাকা উচিত। এর কারণ হলো অ্যানালিটিক্স ইভেন্টটি পাঠানোর উদ্দেশ্য হল ফর্মটি প্রদর্শিত হয়েছে। (এটি ডেভেলপমেন্টে দুইবার চলবে, তবে কীভাবে এটি সামলাতে হয়, তা জানতে [এখানে দেখুন](/learn/synchronizing-with-effects#sending-analytics)।)

তবে, `/api/register` POST অনুরোধটি ফর্মটি _প্রদর্শিত_ হওয়ার কারণে হয় না। আপনি এটি শুধুমাত্র একটি নির্দিষ্ট মুহূর্তে পাঠাতে চান: যখন ব্যবহারকারী বোতামটি চাপেন। এটি শুধু _সেই নির্দিষ্ট ইন্টারঅ্যাকশনে_ ঘটবে। দ্বিতীয় Effect টি মুছে ফেলুন এবং সেই POST অনুরোধটি ইভেন্ট হ্যান্ডলারে সরিয়ে নিন।

```js {12-13}
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic runs because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Good: Event-specific logic is in the event handler
    post('/api/register', { firstName, lastName });
  }
  // ...
}
```

কোনো লজিক ইভেন্ট হ্যান্ডলার বা Effect-এ রাখবেন কি না তা নির্ধারণ করার সময় প্রধান প্রশ্নটি হলো এটি ব্যবহারকারীর দৃষ্টিকোণ থেকে _কিসের লজিক_। যদি এই লজিকটি নির্দিষ্ট একটি ইন্টারঅ্যাকশনের কারণে হয়, তাহলে এটি ইভেন্ট হ্যান্ডলারে রাখুন। আর যদি এটি ব্যবহারকারী কম্পোনেন্টটি স্ক্রিনে _দেখার_ কারণে হয়, তাহলে এটি Effect-এ রাখুন।

### গাণিতিক ক্রম {/*chains-of-computations*/}

কখনো কখনো আপনি একাধিক Effect ব্যবহার করতে প্রলুব্ধ হতে পারেন, যেখানে প্রতিটি Effect অন্য কোনো state এর উপর ভিত্তি করে একটি state সামঞ্জস্য করে:

```js {7-29}
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Avoid: Chains of Effects that adjust the state solely to trigger each other
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

এই কোডে দুটি সমস্যা রয়েছে।

প্রথম সমস্যা হলো এটি খুবই অকার্যকর; প্রতিটি `set` কলের মধ্যে কম্পোনেন্ট (এবং এর চাইল্ড কম্পোনেন্টগুলো) পুনরায় রেন্ডার হতে থাকে। উপরের উদাহরণে, সবচেয়ে খারাপ ক্ষেত্রে (`setCard` → রেন্ডার → `setGoldCardCount` → রেন্ডার → `setRound` → রেন্ডার → `setIsGameOver` → রেন্ডার) এই চেইনটি তিনবার অপ্রয়োজনীয়ভাবে পুনরায় রেন্ডার করে নিচের ট্রীটি।

এটি ধীর না হলেও, কোডটি যখন পরিবর্তিত হয়, তখন আপনি এমন পরিস্থিতিতে পড়বেন যেখানে এই "চেইন"টি নতুন চাহিদার সাথে সামঞ্জস্যপূর্ণ নয়। কল্পনা করুন আপনি গেমের মুভের ইতিহাসের মধ্য দিয়ে পর্যায়ক্রমে এগিয়ে যাওয়ার একটি উপায় যোগ করছেন। আপনি এটি প্রতিটি state ভেরিয়েবলকে অতীতের মানে আপডেট করে করবেন। তবে, অতীতের মানে `card` state সেট করলে Effect চেইনটি পুনরায় চালু হবে এবং আপনার প্রদর্শিত ডেটা পরিবর্তিত হবে। এ ধরনের কোড প্রায়শই কঠোর এবং ভঙ্গুর হয়।

এই ক্ষেত্রে, রেন্ডারিংয়ের সময় যা সম্ভব তা গণনা করাই ভালো এবং event handler এর মধ্যে state সামঞ্জস্য করা উচিত।

```js {6-7,14-26}
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calculate what you can during rendering
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calculate all the next state in the event handler
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

এটি অনেক বেশি কার্যকর। এছাড়াও, যদি আপনি গেমের ইতিহাস দেখার একটি উপায় বাস্তবায়ন করেন, তাহলে এখন আপনি প্রতিটি state ভেরিয়েবলকে অতীতের কোনো মুভে সেট করতে পারবেন এবং প্রতিটি মান পরিবর্তনের জন্য Trigger হওয়া Effect চেইনটি এড়াতে পারবেন। যদি একাধিক event handler-এ লজিক পুনঃব্যবহার করতে হয়, তাহলে আপনি একটি ফাংশন [প্রত্যাহার করে](#sharing-logic-between-event-handlers) সেই handler থেকে কল করতে পারেন।

মনে রাখবেন যে event handler-এর মধ্যে, [state একটি snapshot-এর মতো আচরণ করে।](/learn/state-as-a-snapshot) উদাহরণস্বরূপ, `setRound(round + 1)` কল করার পরেও, `round` ভেরিয়েবলটি সেই মানকে প্রতিফলিত করবে যখন ব্যবহারকারী বোতামটি ক্লিক করেছিলেন। যদি গণনার জন্য পরবর্তী মানের প্রয়োজন হয়, তাহলে এটি ম্যানুয়ালি সংজ্ঞায়িত করুন যেমন `const nextRound = round + 1`।

কিছু ক্ষেত্রে, event handler-এ সরাসরি পরবর্তী state গণনা করা সম্ভব নয়। উদাহরণস্বরূপ, একটি ফর্মের কথা কল্পনা করুন যেখানে একাধিক dropdown রয়েছে এবং পরবর্তী dropdown-এর অপশন আগের dropdown-এর নির্বাচিত মানের উপর নির্ভর করে। তখন, Effects-এর একটি চেইন উপযুক্ত, কারণ আপনি নেটওয়ার্কের সাথে সিঙ্ক্রোনাইজ করছেন।

### এপ্লিকেশন ইনিশিয়েলাইজ করা {/*initializing-the-application*/}

আপনি হয়তো চাইবেন যে কিছু লজিক শুধুমাত্র অ্যাপটি লোড হলে একবারই রান হোক।

আপনি এই কোডটি অ্যাপের শীর্ষ স্তরের কম্পোনেন্টে একটি ইফেক্টে রাখার কথা ভাবতে পারেন।

```js {2-6}
function App() {
  // 🔴 Avoid: Effects with logic that should only ever run once
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

তবে, আপনি দ্রুত বুঝতে পারবেন যে এটি [ডেভেলপমেন্টে দুবার রান করে ](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)। এটি সমস্যার সৃষ্টি করতে পারে—যেমন, হয়তো এটি অথেনটিকেশন টোকেনকে অবৈধ করে ফেলছে কারণ ফাংশনটি দুবার কল করার জন্য ডিজাইন করা হয়নি। সাধারণভাবে, আপনার কম্পোনেন্টগুলো পুনরায় মাউন্ট হতে প্রতিরোধী হওয়া উচিত। এর মধ্যে আপনার শীর্ষ স্তরের `App` কম্পোনেন্টও অন্তর্ভুক্ত।

যদিও প্রোডাকশনে এটি পুনরায় মাউন্ট না-ও হতে পারে, সব কম্পোনেন্টে একই নিয়ম অনুসরণ করলে কোড সরানো এবং পুনঃব্যবহার করা সহজ হয়। যদি কোনো লজিক *অ্যাপ লোড প্রতি একবার* রান করতেই হয় এবং *কম্পোনেন্ট মাউন্ট প্রতি একবার* নয়, তাহলে ট্র্যাক করার জন্য একটি শীর্ষ স্তরের ভেরিয়েবল যোগ করুন যে এটি ইতোমধ্যেই একবার রান হয়েছে কি না।

```js {1,5-6,10}
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

আপনি এটি মডিউল ইনিশিয়ালাইজেশনের সময় এবং অ্যাপ রেন্ডার হওয়ার আগে চালাতে পারেন:

```js {1,5}
if (typeof window !== 'undefined') { // Check if we're running in the browser.
   // ✅ Only runs once per app load
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

শীর্ষ স্তরের কোড একবার রান করে যখন আপনার কম্পোনেন্টটি আমদানি করা হয়—যদিও এটি রেন্ডার না-ও হতে পারে। বিভিন্ন কম্পোনেন্ট আমদানি করার সময় ধীরগতির বা বিস্ময়কর আচরণ এড়াতে, এই প্যাটার্নটি অতিরিক্ত ব্যবহার করবেন না। অ্যাপের সার্বজনীন ইনিশিয়ালাইজেশন লজিককে `App.js` এর মতো রুট কম্পোনেন্ট মডিউলগুলোতে বা আপনার অ্যাপ্লিকেশনের এন্ট্রি পয়েন্টে রাখুন।

### প্যারেন্ট কম্পোনেন্টগুলোকে স্টেট পরিবর্তন সম্পর্কে জানানো {/*notifying-parent-components-about-state-changes*/}

ধরি, আপনি একটি `Toggle` কম্পোনেন্ট লিখছেন যার একটি অভ্যন্তরীণ `isOn` স্টেট আছে যা `true` বা `false` হতে পারে। এটি টগল করার জন্য কিছু ভিন্ন উপায় আছে (ক্লিক করে বা ড্র্যাগ করে)। আপনি চান যে যখনই `Toggle` এর অভ্যন্তরীণ স্টেট পরিবর্তিত হয়, প্যারেন্ট কম্পোনেন্টকে জানানো হোক, তাই আপনি একটি `onChange` ইভেন্ট প্রকাশ করেন এবং একটি ইফেক্ট থেকে এটি কল করেন:

```js {4-7}
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

যেমন আগে বলা হয়েছিল, এটি আদর্শ নয়। `Toggle` প্রথমে তার স্টেট আপডেট করে, এবং তারপর React স্ক্রীনটি আপডেট করে। এর পরে React ইফেক্টটি রান করে, যা প্যারেন্ট কম্পোনেন্ট থেকে প্রাপ্ত `onChange` ফাংশনটি কল করে। এখন প্যারেন্ট কম্পোনেন্ট তার নিজের স্টেট আপডেট করবে, যা একটি নতুন রেন্ডার পাস শুরু করবে। সবকিছু একক পাসে করা আরও ভালো হবে।

ইফেক্টটি মুছে দিন এবং পরিবর্তে একই ইভেন্ট হ্যান্ডলারের মধ্যে *দুইটি* কম্পোনেন্টের স্টেট আপডেট করুন:

```js {5-7,11,16,18}
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ Good: Perform all updates during the event that caused them
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

এই পদ্ধতির মাধ্যমে, `Toggle` কম্পোনেন্ট এবং এর প্যারেন্ট কম্পোনেন্ট উভয়ই ইভেন্টের সময় তাদের স্টেট আপডেট করে। React বিভিন্ন কম্পোনেন্ট থেকে [ব্যাচ আপডেট](/learn/queueing-a-series-of-state-updates) করে, তাই শুধুমাত্র একটি রেন্ডার পাস হবে।

আপনি সম্ভবত সম্পূর্ণ স্টেটটি মুছেও দিতে পারেন, এবং পরিবর্তে প্যারেন্ট কম্পোনেন্ট থেকে `isOn` গ্রহণ করতে পারেন:

```js {1,2}
// ✅ Also good: the component is fully controlled by its parent
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

["স্টেট উপরে তোলা"](/learn/sharing-state-between-components) প্যারেন্ট কম্পোনেন্টকে সম্পূর্ণরূপে `Toggle` নিয়ন্ত্রণ করার সুযোগ দেয় প্যারেন্টের নিজস্ব স্টেট টগল করে। এর মানে হলো প্যারেন্ট কম্পোনেন্টে আরও বেশি লজিক থাকতে হবে, তবে মোট স্টেট কম থাকবে যা নিয়ে চিন্তা করতে হবে। যখনই আপনি দুটি ভিন্ন স্টেট ভেরিয়েবলকে সমন্বয় করতে চেষ্টা করেন, তখন স্টেট উপরে তোলার চেষ্টা করুন!

### পেরন্ট এ ডাটা পাস করা {/*passing-data-to-the-parent*/}

এই `Child` কম্পোনেন্ট কিছু ডেটা Fetch করে এবং তারপর এটি একটি ইফেক্টের মাধ্যমে `Parent` কম্পোনেন্টে পাঠায়:

```js {9-14}
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Avoid: Passing data to the parent in an Effect
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

React-এ, ডেটা প্যারেন্ট কম্পোনেন্ট থেকে তাদের সন্তানের দিকে প্রবাহিত হয়। যখন আপনি স্ক্রীনে কিছু ভুল দেখতে পান, আপনি কোন তথ্য কোথা থেকে এসেছে তা খুঁজে বের করার জন্য কম্পোনেন্ট চেইন উপরে উঠতে পারেন যতক্ষণ না আপনি খুঁজে পান কোন কম্পোনেন্টটি ভুল প্রপ্স পাস করছে বা ভুল স্টেট আছে। যখন সন্তানের কম্পোনেন্টগুলো তাদের প্যারেন্ট কম্পোনেন্টের স্টেট ইফেক্টে আপডেট করে, তখন ডেটার প্রবাহ অনুসরণ করা খুব কঠিন হয়ে পড়ে। যেহেতু সন্তানের এবং প্যারেন্টের উভয়েরই একই ডেটার প্রয়োজন, তাই প্যারেন্ট কম্পোনেন্টকে সেই ডেটা Fetch করতে দিন এবং *সন্তানের কাছে* পাস করুন:

```js {4-5}
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ Good: Passing data down to the child
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

এটি সহজ এবং ডেটার প্রবাহকে পূর্বানুমানযোগ্য রাখে: ডেটা প্যারেন্ট থেকে সন্তানের দিকে প্রবাহিত হয়।

### এক্সটার্নাল স্টোরের জন্য সাবস্ক্রাইব করা {/*subscribing-to-an-external-store*/}

কখনও কখনও, আপনার কম্পোনেন্টগুলোকে React স্টেটের বাইরের কিছু ডেটার জন্য সাবস্ক্রাইব করতে হতে পারে। এই ডেটা একটি তৃতীয়-পক্ষ লাইব্রেরি বা একটি বিল্ট-ইন ব্রাউজার API থেকে আসতে পারে। যেহেতু এই ডেটা React এর জ্ঞানের বাইরে পরিবর্তিত হতে পারে, তাই আপনাকে হাতে হাতে আপনার কম্পোনেন্টগুলোকে এর সাথে সাবস্ক্রাইব করতে হবে। এটি প্রায়ই একটি ইফেক্টের মাধ্যমে করা হয়, উদাহরণস্বরূপ:

```js {2-17}
function useOnlineStatus() {
  // Not ideal: Manual store subscription in an Effect
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

এখানে, কম্পোনেন্টটি একটি এক্সটার্নাল ডেটা স্টোরের জন্য সাবস্ক্রাইব করে (এই ক্ষেত্রে, ব্রাউজারের `navigator.onLine` API)। যেহেতু এই API সার্ভারে বিদ্যমান নেই (এটি প্রাথমিক HTML-এর জন্য ব্যবহার করা যায় না), শুরুতে স্টেট `true` সেট করা হয়। যখনই ব্রাউজারে সেই ডেটা স্টোরের মান পরিবর্তিত হয়, কম্পোনেন্ট তার স্টেট আপডেট করে।

যদিও এটি করার জন্য ইফেক্ট ব্যবহার করা সাধারণ, React-এ একটি বিশেষভাবে ডিজাইন করা হুক আছে যা একটি এক্সটার্নাল স্টোরের জন্য সাবস্ক্রাইব করতে ব্যবহার করা হয়, যা আরও পছন্দসই। ইফেক্টটি মুছে ফেলুন এবং [`useSyncExternalStore`](/reference/react/useSyncExternalStore) কলে প্রতিস্থাপন করুন:

```js {11-16}
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // ✅ Good: Subscribing to an external store with a built-in Hook
  return useSyncExternalStore(
    subscribe, // React won't resubscribe for as long as you pass the same function
    () => navigator.onLine, // How to get the value on the client
    () => true // How to get the value on the server
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

এই পদ্ধতি ইফেক্টের মাধ্যমে মিউটেবল ডেটাকে React স্টেটের সাথে ম্যানুয়ালি সিঙ্ক করার চেয়ে কম ত্রুটিপূর্ণ। সাধারণত, আপনি উপরের মতো একটি কাস্টম হুক লিখবেন যেমন useOnlineStatus() যাতে আপনাকে পৃথক কম্পোনেন্টে এই কোডটি পুনরাবৃত্তি করতে না হয়। [এক্সটার্নাল স্টোরগুলোর জন্য React কম্পোনেন্ট থেকে সাবস্ক্রাইব করার বিষয়ে আরও পড়ুন।](/reference/react/useSyncExternalStore)

### Fetching data {/*fetching-data*/}

অনেক অ্যাপ্লিকেশন ডেটা Fetch করার জন্য ইফেক্ট ব্যবহার করে। একটি ডেটা Fetch করার ইফেক্ট এইভাবে লেখা খুব সাধারণ:

```js {5-10}
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 Avoid: Fetching without cleanup logic
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

আপনাকে এই ফেচটি একটি ইভেন্ট হ্যান্ডলারে স্থানান্তর করতে *হবেনা*।

এটি প্রথম উদাহরণগুলোর সাথে একটি বৈপরীত্য মনে হতে পারে যেখানে আপনাকে লজিকটি ইভেন্ট হ্যান্ডলারগুলিতে রাখতে হয়েছে! তবে, মনে রাখবেন যে এটি *টাইপিং ইভেন্ট* নয় যা Fetch করার প্রধান কারণ। সার্চ ইনপুটগুলি প্রায়শই URL থেকে পূর্বনির্ধারিত হয়, এবং ব্যবহারকারী ইনপুটটি স্পর্শ না করেই ব্যাক এবং ফরোয়ার্ডে নেভিগেট করতে পারে।

`page` এবং `query` কোথা থেকে আসছে তা গুরুত্বপূর্ণ নয়। যখন এই কম্পোনেন্টটি দৃশ্যমান থাকে, আপনি `results` কে বর্তমান `page` এবং `query` এর জন্য নেটওয়ার্কের ডেটার সাথে [সিঙ্ক্রোনাইজ](/learn/synchronizing-with-effects) রাখতে চান। এ কারণেই এটি একটি ইফেক্ট।

তবে, উপরের কোডে একটি বাগ আছে। কল্পনা করুন আপনি দ্রুত `"hello"` টাইপ করছেন। তারপর `query` পরিবর্তিত হবে `"h"` থেকে `"he"`, `"hel"`, `"hell"` এবং `"hello"`-তে। এটি আলাদা ফেচগুলি শুরু করবে, কিন্তু প্রতিক্রিয়াগুলির কোন অর্ডারে আসবে তা নিশ্চিত নয়। উদাহরণস্বরূপ, `"hell"` প্রতিক্রিয়া আসতে পারে *এরপর* `"hello"` প্রতিক্রিয়ার। যেহেতু এটি শেষবার `setResults()` কল করবে, আপনি ভুল সার্চ ফলাফল প্রদর্শন করবেন। একে বলা হয় ["রেস কন্ডিশন"](https://en.wikipedia.org/wiki/Race_condition): দুটি ভিন্ন অনুরোধ "রেস" করেছে একে অপরের বিরুদ্ধে এবং প্রত্যাশিত অর্ডারের চেয়ে ভিন্ন অর্ডারে এসেছে।

**রেস কন্ডিশনটি ঠিক করতে, আপনাকে [একটি ক্লিনআপ ফাংশন](/learn/synchronizing-with-effects#fetching-data) যোগ করতে হবে যাতে পুরানো প্রতিক্রিয়াগুলি উপেক্ষা করা হয়:**

```js {5,7,9,11-13}
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

এটি নিশ্চিত করে যে যখন আপনার ইফেক্ট ডেটা ফেচ করে, তখন শেষ রিকোয়েস্ট করা রেসপন্স ছাড়া সব রিকোয়েস্ট উপেক্ষা করা হবে।

ডেটা ফেচিং বাস্তবায়নের সাথে রেস কন্ডিশনগুলি পরিচালনা করা একমাত্র চ্যালেঞ্জ নয়। আপনি রেসপন্সগুলির ক্যাশিং সম্পর্কেও চিন্তা করতে চাইতে পারেন (যাতে ব্যবহারকারী ব্যাক ক্লিক করলে আগের স্ক্রিনটি তাত্ক্ষণিকভাবে দেখতে পারে), সার্ভারে ডেটা কীভাবে ফেচ করতে হবে (যাতে প্রাথমিক সার্ভার-রেন্ডার করা HTML-এ ফেচ করা কনটেন্ট থাকে স্পিনারের পরিবর্তে), এবং নেটওয়ার্ক ওয়াটারফলগুলি কীভাবে এড়ানো যায় (যাতে একটি সন্তান ডেটা ফেচ করতে পারে প্রতিটি প্যারেন্টের জন্য অপেক্ষা না করে)।

**এই সমস্যাগুলি যেকোন UI লাইব্রেরিতে প্রযোজ্য, শুধু React নয়। এগুলি সমাধান করা সহজ নয়, এজন্য আধুনিক [ফ্রেমওয়ার্কগুলি](/learn/start-a-new-react-project#production-grade-react-frameworks) ইফেক্টে ডেটা ফেচ করার চেয়ে আরও কার্যকর বিল্ট-ইন ডেটা ফেচিং যান্ত্রিক সরবরাহ করে।**

যদি আপনি একটি ফ্রেমওয়ার্ক ব্যবহার না করেন (এবং আপনার নিজস্ব তৈরি করতে না চান) তবে ইফেক্ট থেকে ডেটা ফেচিংকে আরও ব্যবহারকারী বান্ধব করতে চান, তবে এই উদাহরণের মতো আপনার ফেচিং লজিকটিকে একটি কাস্টম হুকের মধ্যে নিষ্কাশন করতে বিবেচনা করুন:

```js {4}
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
```

আপনি সম্ভবত ত্রুটি পরিচালনার জন্য কিছু লজিক যুক্ত করতে চান এবং কন্টেন্ট লোড হচ্ছে কিনা তা ট্র্যাক করতে চান। আপনি নিজেই এমন একটি হুক তৈরি করতে পারেন অথবা React ইকোসিস্টেমে ইতিমধ্যেই উপলব্ধ অনেক সমাধানের মধ্যে একটি ব্যবহার করতে পারেন। **যদিও এটি একা একটি ফ্রেমওয়ার্কের বিল্ট-ইন ডেটা ফেচিং যান্ত্রিকের চেয়ে ততটা কার্যকর হবে না, একটি কাস্টম হুকে ডেটা ফেচিং লজিকটি স্থানান্তর করা পরবর্তীতে একটি কার্যকর ডেটা ফেচিং কৌশল গ্রহণ করা সহজ করবে।**

সাধারণভাবে, যখনই আপনাকে ইফেক্ট লিখতে বাধ্য হতে হয়, তখন নজর রাখুন কখন আপনি একটি কাস্টম হুকে একটি কার্যকরী অংশ নিষ্কাশন করতে পারেন একটি আরও ঘোষণামূলক এবং উদ্দেশ্য-নির্মিত API এর সাথে যেমন উপরে `useData`। আপনার কম্পোনেন্টগুলিতে যত কম কাঁচা `useEffect` কল থাকবে, আপনার অ্যাপ্লিকেশনটি বজায় রাখা ততটাই সহজ হবে।

<Recap>

- যদি আপনি রেন্ডার করার সময় কিছু হিসাব করতে পারেন, তবে আপনাকে একটি ইফেক্টের প্রয়োজন নেই।
- ব্যয়বহুল গণনা ক্যাশ করার জন্য, `useEffect` এর পরিবর্তে `useMemo` যুক্ত করুন।
- একটি সম্পূর্ণ কম্পোনেন্ট গাছের স্টেট রিসেট করতে, এর জন্য একটি ভিন্ন `key` পাস করুন।
- একটি প্রোপ পরিবর্তনের প্রতিক্রিয়া হিসাবে একটি নির্দিষ্ট স্টেটের অংশ রিসেট করতে, এটি রেন্ডারিংয়ের সময় সেট করুন।
- কোড যা একটি কম্পোনেন্ট প্রদর্শিত হওয়ার কারণে চালিত হয় তা ইফেক্টে থাকা উচিত, বাকি অংশ ইভেন্টে থাকা উচিত।
- যদি আপনাকে কয়েকটি কম্পোনেন্টের স্টেট আপডেট করতে হয়, তবে এটি একটি একক ইভেন্টের মধ্যে করা ভালো।
- যখনই আপনি বিভিন্ন কম্পোনেন্টে স্টেট ভেরিয়েবলগুলিকে সিঙ্ক্রোনাইজ করতে চেষ্টা করেন, তাহলে স্টেটকে আপ করতে বিবেচনা করুন।
- আপনি ইফেক্টের মাধ্যমে ডেটা ফেচ করতে পারেন, তবে রেস কন্ডিশন এড়ানোর জন্য ক্লিনআপ বাস্তবায়ন করতে হবে।

</Recap>

<Challenges>

#### এফেক্ট ছাড়া ডাটা ট্রান্সফর্ম {/*transform-data-without-effects*/}

নিচের `TodoList` একটি টোডোর তালিকা প্রদর্শন করে। যখন "শুধুমাত্র সক্রিয় টু-ডো দেখান" চেকবক্সটি টিক দেওয়া হয়, সম্পন্ন টোডোগুলি তালিকায় প্রদর্শিত হয় না। যে কোন টোডো দৃশ্যমান হোক না কেন, ফুটারে এখনও সম্পন্ন না হওয়া টোডোগুলির সংখ্যা প্রদর্শিত হয়।

এই কম্পোনেন্টটি সমস্ত অপ্রয়োজনীয় স্টেট এবং ইফেক্ট মুছে ফেলে সরল করুন।

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [activeTodos, setActiveTodos] = useState([]);
  const [visibleTodos, setVisibleTodos] = useState([]);
  const [footer, setFooter] = useState(null);

  useEffect(() => {
    setActiveTodos(todos.filter(todo => !todo.completed));
  }, [todos]);

  useEffect(() => {
    setVisibleTodos(showActive ? activeTodos : todos);
  }, [showActive, todos, activeTodos]);

  useEffect(() => {
    setFooter(
      <footer>
        {activeTodos.length} todos left
      </footer>
    );
  }, [activeTodos]);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      {footer}
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

<Hint>

যদি আপনি রেন্ডার করার সময় কিছু গণনা করতে পারেন, তবে আপনাকে এমন স্টেট বা একটি ইফেক্টের প্রয়োজন নেই যা এটি আপডেট করে।

</Hint>

<Solution>

এই উদাহরণে দুটি মূল স্টেট রয়েছে: `todos` এর তালিকা এবং `showActive` স্টেট ভেরিয়েবল যা চেকবক্সটি টিক দেওয়া হয়েছে কিনা তা প্রতিনিধিত্ব করে। অন্যান্য সমস্ত স্টেট ভেরিয়েবল [অপ্রয়োজনীয়](/learn/choosing-the-state-structure#avoid-redundant-state) এবং রেন্ডার করার সময় গণনা করা যেতে পারে। এর মধ্যে `footer` অন্তর্ভুক্ত যা আপনি সরাসরি চারপাশের JSX-তে স্থানান্তর করতে পারেন।

আপনার ফলাফলটি এরকম হবে:

<Sandpack>

```js
import { useState } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
      <footer>
        {activeTodos.length} todos left
      </footer>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

</Solution>

#### এফেক্ট ছড়া ক্যালকুলেশ ক্যাশ করা {/*cache-a-calculation-without-effects*/}

এই উদাহরণে, টোডোগুলিকে ফিল্টার করার জন্য একটি পৃথক ফাংশন `getVisibleTodos()` এ স্থানান্তরিত করা হয়েছে। এই ফাংশনের মধ্যে একটি `console.log()` কল রয়েছে যা আপনাকে দেখাতে সাহায্য করে কখন এটি কল হচ্ছে। "শুধুমাত্র সক্রিয় টোডো দেখান" টগল করলে লক্ষ্য করুন যে এটি `getVisibleTodos()` পুনরায় চালানোর কারণ হয়। এটি প্রত্যাশিত কারণ আপনি কোন টোডোগুলি প্রদর্শন করবেন তা টগল করলে দৃশ্যমান টোডোগুলি পরিবর্তিত হয়।

আপনার কাজ হল `TodoList` কম্পোনেন্টে `visibleTodos` তালিকা পুনঃগণনা করা ইফেক্টটি মুছে ফেলা। তবে, আপনাকে নিশ্চিত করতে হবে যে `getVisibleTodos()` *পুনরায় চালানো হয় না* (এবং তাই কোনও লগ মুদ্রণ করে না) যখন আপনি ইনপুটে টাইপ করেন।

<Hint>

একটি সমাধান হল দৃশ্যমান টোডোগুলিকে ক্যাশ করার জন্য একটি `useMemo` কল যুক্ত করা। একটি অন্য, কম স্পষ্ট সমাধানও রয়েছে।

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const [visibleTodos, setVisibleTodos] = useState([]);

  useEffect(() => {
    setVisibleTodos(getVisibleTodos(todos, showActive));
  }, [todos, showActive]);

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() was called ${++calls} times`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

<Solution>

স্টেট ভেরিয়েবল এবং ইফেক্টটি মুছে ফেলুন, এবং পরিবর্তে `getVisibleTodos()` কল করার ফলাফল ক্যাশ করার জন্য একটি `useMemo` কল যুক্ত করুন:

<Sandpack>

```js
import { useState, useMemo } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const [text, setText] = useState('');
  const visibleTodos = useMemo(
    () => getVisibleTodos(todos, showActive),
    [todos, showActive]
  );

  function handleAddClick() {
    setText('');
    setTodos([...todos, createTodo(text)]);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() was called ${++calls} times`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

এই পরিবর্তনের সাথে, `getVisibleTodos()` কেবল তখনই কল হবে যদি `todos` অথবা `showActive` পরিবর্তিত হয়। ইনপুটে টাইপ করা কেবল `text` স্টেট ভেরিয়েবল পরিবর্তন করে, তাই এটি `getVisibleTodos()` কে কল করতে ট্রিগার করে না।

আরও একটি সমাধান রয়েছে যা `useMemo` এর প্রয়োজন নেই। যেহেতু `text` স্টেট ভেরিয়েবল সম্ভবত টোডোর তালিকাকে প্রভাবিত করতে পারে না, আপনি `NewTodo` ফর্মটিকে একটি পৃথক কম্পোনেন্টে স্থানান্তরিত করতে পারেন এবং `text` স্টেট ভেরিয়েবলটিকে এর ভিতরে নিয়ে যেতে পারেন:

<Sandpack>

```js
import { useState, useMemo } from 'react';
import { initialTodos, createTodo, getVisibleTodos } from './todos.js';

export default function TodoList() {
  const [todos, setTodos] = useState(initialTodos);
  const [showActive, setShowActive] = useState(false);
  const visibleTodos = getVisibleTodos(todos, showActive);

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={showActive}
          onChange={e => setShowActive(e.target.checked)}
        />
        Show only active todos
      </label>
      <NewTodo onAdd={newTodo => setTodos([...todos, newTodo])} />
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ? <s>{todo.text}</s> : todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}

function NewTodo({ onAdd }) {
  const [text, setText] = useState('');

  function handleAddClick() {
    setText('');
    onAdd(createTodo(text));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAddClick}>
        Add
      </button>
    </>
  );
}
```

```js src/todos.js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
  console.log(`getVisibleTodos() was called ${++calls} times`);
  const activeTodos = todos.filter(todo => !todo.completed);
  const visibleTodos = showActive ? activeTodos : todos;
  return visibleTodos;
}

export function createTodo(text, completed = false) {
  return {
    id: nextId++,
    text,
    completed
  };
}

export const initialTodos = [
  createTodo('Get apples', true),
  createTodo('Get oranges', true),
  createTodo('Get carrots'),
];
```

```css
label { display: block; }
input { margin-top: 10px; }
```

</Sandpack>

এই পদ্ধতিটি প্রয়োজনীয়তা মেটায়। যখন আপনি ইনপুটে টাইপ করেন, কেবল `text` স্টেট ভেরিয়েবল আপডেট হয়। যেহেতু `text` স্টেট ভেরিয়েবলটি সন্তানের `NewTodo` কম্পোনেন্টে আছে, তাই অভিভাবক `TodoList` কম্পোনেন্টটি পুনরায় রেন্ডার হবে না। এ কারণেই আপনি টাইপ করার সময় `getVisibleTodos()` কল হয় না। (যদি `TodoList` অন্য কোনও কারণে পুনরায় রেন্ডার হয় তবে এটি এখনও কল হবে।)

</Solution>

#### ইফেক্ট ছাড়া স্টেট রিসেট করা {/*reset-state-without-effects*/}

এই `EditContact` কম্পোনেন্টটি `{ id, name, email }` আকারের একটি যোগাযোগ অবজেক্ট `savedContact` প্রপ হিসেবে গ্রহণ করে। নাম এবং ইমেইল ইনপুট ফিল্ডগুলি সম্পাদনা করার চেষ্টা করুন। আপনি যখন সেভ চাপেন, ফর্মের উপরে যোগাযোগের বাটনটি সম্পাদিত নামের সাথে আপডেট হয়। আপনি যখন রিসেট চাপেন, ফর্মে কোনও চলমান পরিবর্তন বাতিল হয়ে যায়। এটি অনুভব করার জন্য এই UI-তে খেলা করুন।

আপনি যখন উপরের বাটনগুলির সাহায্যে একটি যোগাযোগ নির্বাচন করেন, ফর্মটি সেই যোগাযোগের বিস্তারিত তথ্য প্রতিফলিত করতে রিসেট হয়। এটি `EditContact.js` এর মধ্যে একটি ইফেক্টের মাধ্যমে করা হয়। এই ইফেক্টটি মুছে ফেলুন। `savedContact.id` পরিবর্তিত হলে ফর্মটি রিসেট করার জন্য আরেকটি উপায় খুঁজুন।

<Sandpack>

```js src/App.js hidden
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        savedContact={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js hidden
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js active
import { useState, useEffect } from 'react';

export default function EditContact({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  useEffect(() => {
    setName(savedContact.name);
    setEmail(savedContact.email);
  }, [savedContact]);

  return (
    <section>
      <label>
        Name:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Save
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
      }}>
        Reset
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Hint>

এটি খুবই ভালো হবে যদি কোনও উপায় থাকত যা রিঅ্যাক্টকে বলতে পারে যে `savedContact.id` পরিবর্তিত হলে `EditContact` ফর্মটি ধারণাগতভাবে একটি _বিভিন্ন যোগাযোগের ফর্ম_ এবং এটি স্টেট সংরক্ষণ করা উচিত নয়। আপনি কি এমন কোনও উপায় মনে করতে পারেন?

</Hint>

<Solution>

`EditContact` কম্পোনেন্টটিকে দুটি অংশে ভাগ করুন। সমস্ত ফর্ম স্টেটকে অভ্যন্তরীণ `EditForm` কম্পোনেন্টে স্থানান্তরিত করুন। বাইরের `EditContact` কম্পোনেন্টটি এক্সপোর্ট করুন এবং এটি অভ্যন্তরীণ `EditForm` কম্পোনেন্টকে `savedContact.id` কে `key` হিসেবে পাঠাতে বলুন। এর ফলস্বরূপ, অভ্যন্তরীণ `EditForm` কম্পোনেন্টটি যখন আপনি একটি ভিন্ন যোগাযোগ নির্বাচন করেন, তখন সমস্ত ফর্ম স্টেট রিসেট করে এবং DOM পুনরায় তৈরি করে।

<Sandpack>

```js src/App.js hidden
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        savedContact={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js hidden
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js active
import { useState } from 'react';

export default function EditContact(props) {
  return (
    <EditForm
      {...props}
      key={props.savedContact.id}
    />
  );
}

function EditForm({ savedContact, onSave }) {
  const [name, setName] = useState(savedContact.name);
  const [email, setEmail] = useState(savedContact.email);

  return (
    <section>
      <label>
        Name:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: savedContact.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Save
      </button>
      <button onClick={() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
      }}>
        Reset
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

</Solution>

#### এফেক্ট ছাড়া ফর্ম সাবমিট করা {/*submit-a-form-without-effects*/}

এই `Form` উপাদানটি আপনাকে একটি বন্ধুদের কাছে বার্তা পাঠানোর সুযোগ দেয়। যখন আপনি ফর্মটি জমা দেন, `showForm` রাষ্ট্র পরিবর্তনশীলটি `false` সেট করা হয়। এটি একটি প্রভাবকে সক্রিয় করে যা `sendMessage(message)` কল করে, যা বার্তাটি পাঠায় (আপনি এটি কনসোলে দেখতে পাবেন)। বার্তা পাঠানোর পরে, আপনি একটি "ধন্যবাদ" সংলাপ দেখবেন যার একটি "চ্যাট খুলুন" বোতাম রয়েছে যা আপনাকে আবার ফর্মে ফিরিয়ে নিয়ে যায়।

আপনার অ্যাপের ব্যবহারকারীরা অনেক বেশি বার্তা পাঠাচ্ছেন। চ্যাটিংকে কিছুটা আরও কঠিন করার জন্য, আপনি সিদ্ধান্ত নিয়েছেন যে "ধন্যবাদ" সংলাপটি *প্রথমে* প্রদর্শন করবেন ফর্মের পরিবর্তে। এই পরিবর্তনটি করার সাথে সাথেই `showForm` রাষ্ট্র পরিবর্তনশীলটিকে `true` থেকে `false`তে সেট করুন। যখন আপনি এই পরিবর্তনটি করবেন, কনসোলে একটি খালি বার্তা পাঠানো হয়েছে এমন একটি বার্তা প্রদর্শিত হবে। এই লজিকে কিছু ভুল রয়েছে!

সমস্যার মূল কারণ কী? এবং আপনি এটি কিভাবে ঠিক করতে পারেন?

<Hint>

Should the message be sent _because_ the user saw the "Thank you" dialog? Or is it the other way around ব্যবহারকারী "ধন্যবাদ" দেখার _কারণে_ মেসেজটি পাঠানো উচিত কি? নাকি উল্টোটা?

</Hint>

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Form() {
  const [showForm, setShowForm] = useState(true);
  const [message, setMessage] = useState('');

  useEffect(() => {
    if (!showForm) {
      sendMessage(message);
    }
  }, [showForm, message]);

  function handleSubmit(e) {
    e.preventDefault();
    setShowForm(false);
  }

  if (!showForm) {
    return (
      <>
        <h1>Thanks for using our services!</h1>
        <button onClick={() => {
          setMessage('');
          setShowForm(true);
        }}>
          Open chat
        </button>
      </>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit" disabled={message === ''}>
        Send
      </button>
    </form>
  );
}

function sendMessage(message) {
  console.log('Sending message: ' + message);
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

<Solution>

`showForm` রাষ্ট্র পরিবর্তনশীলটি ফর্ম বা "ধন্যবাদ" সংলাপটি প্রদর্শন করা হবে কিনা তা নির্ধারণ করে। তবে, আপনি বার্তাটি পাঠাচ্ছেন না কারণ "ধন্যবাদ" সংলাপটি _প্রদর্শিত_ হয়েছে। আপনি বার্তা পাঠাতে চান কারণ ব্যবহারকারী _ফর্মটি জমা দিয়েছে।_ বিভ্রান্তিকর প্রভাবটি মুছে ফেলুন এবং `sendMessage` কলটিকে `handleSubmit` ইভেন্ট হ্যান্ডলারের ভিতরে স্থানান্তর করুন:

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Form() {
  const [showForm, setShowForm] = useState(true);
  const [message, setMessage] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    setShowForm(false);
    sendMessage(message);
  }

  if (!showForm) {
    return (
      <>
        <h1>Thanks for using our services!</h1>
        <button onClick={() => {
          setMessage('');
          setShowForm(true);
        }}>
          Open chat
        </button>
      </>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit" disabled={message === ''}>
        Send
      </button>
    </form>
  );
}

function sendMessage(message) {
  console.log('Sending message: ' + message);
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

দেখুন, এই সংস্করণে, শুধুমাত্র _ফর্মটি জমা দেওয়া_ (যা একটি ইভেন্ট) বার্তা পাঠানোর কারণ হয়। এটি `showForm` যদি প্রাথমিকভাবে `true` বা `false` হিসাবে সেট করা হয়, উভয় ক্ষেত্রেই সমানভাবে কাজ করে। (এটি `false` এ সেট করুন এবং দেখুন অতিরিক্ত কনসোল বার্তা নেই।)

</Solution>

</Challenges>
