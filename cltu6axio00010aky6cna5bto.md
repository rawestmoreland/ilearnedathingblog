---
title: "I Built a Habit Tracking App with WatermelonDB and React Native"
seoTitle: "I built a habit tracking app based on the Law of 100"
seoDescription: "I learned a lot about watermelonDB and building a local-first React Native app. After a lot of trial and error I finally got the library installed."
datePublished: Sat Mar 16 2024 14:18:26 GMT+0000 (Coordinated Universal Time)
cuid: cltu6axio00010aky6cna5bto
slug: i-built-a-habit-tracking-app-with-watermelondb-and-react-native
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/PkbZahEG2Ng/upload/85a71b0c36f97c29007bd9b9629609d8.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1710598609376/f7813359-cc29-4d79-b2a9-75924fe92eb8.jpeg
tags: react-native, sqlite, watermelondb

---

Meet Hundred Habits - a habit-tracking app I recently published on both the [iOS App Store](https://apps.apple.com/us/app/hundred-habits/id6478966966) and the [Google Play Store](https://play.google.com/store/apps/details?id=com.westmorelandcreative.hundredhabits). Hundred Habits is based on the Law of 100 principle. If you're not familiar - the Law of 100 is a commitment to a task or a goal for 100 iterations or days. It's about doing something consistently, without fail, 100 times.

My mobile development framework of choice is React Native. I was originally a huge Flutter advocate when the platform was first introduced several years ago, but given that I'm primarily a Javascript / React developer, it makes more sense for me to keep my brain focused on that stack. So React Native it is.

Hundred Habits is simple. There's no auth. There's no account creation. Everything is stored locally on the user's device. It's simple enough that everything could probably be stored in local storage as JSON strings, but I wanted to play around with WatermelonDB and SQLite.

### Why Watermelon?

Why not just use SQLite directly? What appeals most about WatermelonDB to me is the *speed*. The number one goal of Watermelon is **real-world performance**. Watermelon is *fast*. Loading a full database of tens of thousands of records onto a mobile device is expensive. According to [their docs](https://watermelondb.dev/docs#why-watermelon), Watermelon fixes this by **being lazy**. Nothing is loaded until it's requested.

Another huge advantage of Watermelon is that it's **fully observable**. Whenever you change a record, all UI that depends on that record will automatically re-render. (Well... *almost* automatically). You just need to wrap your components in a way that allows them to observe changes.

```javascript
const Comment = ({ comment }) => (
  <View style={styles.commentBox}>
    <Text>{comment.body} — by {comment.author}</Text>
  </View>
)

// This is how you make your app reactive! ✨
const enhance = withObservables(['comment'], ({ comment }) => ({
  comment,
}))
const EnhancedComment = enhance(Comment)
```

### Challenges

I did run into a few challenges while setting up the project. The first big challenge was that I use [Expo](https://expo.dev) for my React Native apps. The Watermelon docs don't really have any Expo-centric instructions so I had to do some googling. Luckily, [this plugin](https://github.com/morrowdigital/watermelondb-expo-plugin) exists, which makes configuring Watermelon in your React Native Expo project a breeze.

It's as simple as installing the plugin and then adding it to your `app.json` .

Install:

```plaintext
yarn add @morrowdigital/watermelondb-expo-plugin
```

Add the plugin to your app:

```json
{
  "plugins": [
      [
        "@morrowdigital/watermelondb-expo-plugin"
      ],
      [
        "expo-build-properties",
        {
          "android": {
            "kotlinVersion": "1.6.10",
            "packagingOptions": {
              "pickFirst": [
                "**/libc++_shared.so"
              ]
            }
          }
        }
      ]
  ]
}
```

After that, I had to run a `npx expo run:ios` and a `npx expo run:android` in order to get all of the native libraries installed. Adding custom native code is covered in [this guide](https://docs.expo.dev/workflow/customizing/) from Expo.

The people at MorrowDigital also have [this great article](https://www.themorrow.digital/blog/how-to-use-watermelondb-with-react-native-expo) for getting everything set up.

Here's my [github repo](https://github.com/rawestmoreland/100-days) for this project. If you have any question, please comment below!