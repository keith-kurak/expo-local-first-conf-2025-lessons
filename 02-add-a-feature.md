# Module 01: Let's build with Expo and Livestore

### Goal

Let's hone in on a thin vertical slice of functionality across Livestore schema and React Native UI development to get a small taste of what it's like working on such a project.

### Concepts

- Livestore schema changes
- React Native frontend development
- React Native Reanimated (native animations)

### Tasks

- Add the schema to designate a reaction as a "super" reaction
- Add a migration for the schema
- Add the long-press-for-super-reaction functionality
- Give this functionality a fancy animation using Reanimated

### Useful links

- [How Livestore deals with schema changes](https://livestore.dev/reference/rules-of-mutations/)

# Exercises

## Exercise 1: Super-react schema

The app already has "reactions", where you can give a note a thumbs up or a heart. Let's add "super reaction" functionality, where holding down the reaction will make it "super." We'll eventually add a fancy animation for this, but for now, let's focus on the data layer.

We're going to add a single field to the `reaction` table, a `type` that is either `regular` (the default) or `super`. Long-pressing on the initial reaction will make it super. Long-pressing on an existing reaction you've already made will update that reaction to be super.

When we think about databases and changing schema, we often think about schema migrations and migrations. In the case of Livestore, we only make schema changes that are forward-compatible. We can add fields as long as they're optional or have a default value. That way, pre-super-reaction events can be processed by a post-super-reaction client version (they're just default to "regular").

1. Inside **shared/schema.ts**, in the `reaction` table, under `columns`, add the `type` field:

```diff
deletedAt: State.SQLite.integer({
  nullable: true,
  schema: Schema.DateFromNumber,
}),
+ type: State.SQLite.text({schema: Schema.Literal("regular", "super"), default: "regular",}),
```

2. In the same file, look at the `materializers`. These are the mutations- the events that make up the change history in Livestore. We need to add a new materializer that will create a note with a `type`.

Notice that the materializers so far have a `v1` in front. Once a materializer is present, it cannot be deleted, or else it would not be possible to process paste events. So, it can be helpful to create a new `v2` materializer that incorporates the `type` field. Add the following to the `materializers` list:

```ts
"v2.NoteReacted": ({ id, noteId, emoji, type, createdBy }) =>
  reaction.insert({
    id,
    noteId,
    emoji,
    type: type as "regular" | "super",
    createdBy,
    createdAt: new Date(),
  }),
```

3. You'll probably notice some TypeScript squiggles at this point. This is because we can't just define the materializer without a corresponding event. Head over to **events.ts**. Add a new event for the v2 reaction:

```ts
export const noteReacted = Events.synced({
  name: "v2.NoteReacted",
  schema: Schema.Struct({
    id: Schema.String,
    noteId: Schema.String,
    emoji: Schema.String,
    type: Schema.Literal("regular", "super"),
    createdBy: Schema.String,
  }),
});
```

You'll see more squiggles now, because there's already an event named `noteReacted` being exported. This export is useful for when we actually implement the operation in the UI. Let's rename the old `noteReacted` to `noteReacted_v1`. This means that the old event still exists for the sake of data created before the super like implementation, but the event that's actually used in the UI going forward will always be the new one.

```diff
- export const noteReacted = Events.synced({
+ export const noteReacted_v1 = Events.synced({
```

Let's update the bottom sheet that pops up when you'd like to add a reaction to accept the new type parameter. This also serves as a bit of an example of how file-based routing works in an Expo app. The bottom sheet is in **packages/mobile/app/(home)/reaction/[note].tsx**. If you look inside the **packages/mobile/app/(home)/\_layout.tsx** file, you'll see the `reaction/[note]` route referenced with `presentation: "formSheet"`. This means that the bottom sheet is a separate screen that pushes on top of the list of notes, but is partially transparent, so you can still see the list underneath it. The `[note]` part of the route is where the note ID is passed, so the screen knows which note the reaction is for.

4. In **app/(home)/reaction/[note].tsx**, update `handleReaction` to incorporate the type:

```diff
- function handleReaction(emoji: string) {
+ function handleReaction(emoji: string, type: "regular" | "super" = "regular") {
  store.commit(
    events.noteReacted({
      id: nanoid(),
      noteId: noteId,
      emoji: emoji,
+      type: type,
      createdBy: user!.name,
    })
  );
  router.back();
}
```

> If you hover over the `noteCreated` function, you'll see in intellisense how it is mapping back to `v1.noteCreated`, but it omits the v1.

Then update the `onPress` and add the `onLongPress` events:

```diff
<Pressable
  key={reaction}
+  onLongPress={() => handleReaction(reaction, "super")}
>
  <ReactionItem key={reaction} reaction={reaction} />
</Pressable>
```

🏃**Try it.** You should be able to regular react and super react now. Though you can't see the effects of a super-react yet. You can inspect the event in devtools in order to know that the super reaction was included as part of the note creation event.

## Exercise 2: Showing the super reaction

There's a lot of different ways we could show super reactions to differentiate them from regular reactions. We could show them as separate icons, or we could show separate badge numbers.

Let's do something really simple for now, and highlight the reaction in yellow if there's at least one super reaction for it. This will involve us creating a new query to identify this condition.

1. In **packages/shared/queries.ts**, add this query for just super reactions:

```ts
export const noteReactionsSuper$ = (noteId: string) =>
  queryDb(
    tables.reaction.where({
      noteId,
      type: "super",
      deletedAt: null,
    }),
    { label: `super-reactions-${noteId}` }
  );
```

2. In **packages/mobile/components/NoteReaction.tsx**, add a hook to run the above query:

```tsx
const superReactions = useQuery(noteReactionsSuper$(noteId));
```

3. Add styling to each emoji to change its color once its super:

```diff
{Object.entries(reactionCounts).map(([emoji, count]) => (
  <Pressable
    key={emoji}
-    style={noteReactionsStyles.reactionButton as ViewStyle}
+    style={[noteReactionsStyles.reactionButton as ViewStyle, superReactions.find(sr => sr.emoji === emoji) && { backgroundColor: 'yellow' }]}
```

🏃**Try it.** Super reactions should now change the how the reactions look.

## Exercise 3: Modifying an existing reaction

Let's now add the functionality to long-press on an existing reaction to make it super.

In **/packages/mobile/components/NoteReactions.tsx**, add an `onLongPress` property to the reaction pressable component.

```diff
 <Pressable
    key={emoji}
    style={noteReactionsStyles.reactionButton as ViewStyle}
+    onLongPress={() => {
+      store.commit(
+        events.noteReacted({
+          id: nanoid(),
+          noteId,
+          emoji,
+          type: "super",
+          createdBy: user!.name,
+        })
+      );
+    }}
    onPress={() => {
    ...
```

If you’re feeling fancy, consider adding haptic feedback to enhance the user experience.

🏃**Try it.** You should be able to create both regular reactions and super reactions now. Once we deploy the sync server, you can try it on the Expo Go app to test the haptic feedback.

## Exercise 4: Fancy animations!

Super reactions aren't really all that super until they have some pizzaz. We're going to add a cool particle effect using React Native Reanimated. This will occur while the long press is in progress, like you're charging up until the reaction is "super".

Open **/packages/mobile/components/ReactionParticles.tsx**, take a look at this component. It creates a particle explosion effect using React Native Reanimated. The component generates 8 small animated particles that burst outward from a central point in a circular pattern. Each particle has randomized properties (size, delay, distance) and uses shared values with animated styles to control its opacity, scale, and position. The particles appear, move outward, and then fade away, creating a satisfying visual feedback effect when a reaction becomes "super". We'll use this to add visual flair to our super reactions.

In **/packages/mobile/components/NoteReactions.tsx**,

1. First, let's add a state variable to keep track of when to show or hide the particles:

```tsx
const [activeParticleEmoji, setActiveParticleEmoji] = useState<string | null>(
  null
);
```

2. Let's also create a function to handle showing the particles for the specific emoji. This will add a timeout of 1 second to reset the animation:

```tsx
function handleShowParticles(emoji: string) {
  setActiveParticleEmoji(emoji);
  setTimeout(() => {
    setActiveParticleEmoji(null);
  }, 1000);
}
```

3. Now, modify the `onLongPress` handler to call this function:

```diff
 <Pressable
    key={emoji}
    style={[noteReactionsStyles.reactionButton as ViewStyle, superReactions.find(sr => sr.emoji === emoji) && { backgroundColor: 'yellow' }]}
    onLongPress={() => {
+     handleShowParticles(emoji);
      store.commit(
        events.noteReacted({
          id: nanoid(),
          noteId,
          emoji,
          type: "super",
          createdBy: user!.name,
        })
      );
    }}
    onPress={() => {
    ...
```

4. Finally, render the particles component only when the `activeParticleEmoji` is equal to the current emoji being super-reacted:

```tsx
{
  activeParticleEmoji === emoji && (
    <ReactionParticles
      color={reactionColors[Math.floor(Math.random() * reactionColors.length)]}
    />
  );
}
```

This should be placed within the Pressable component, right before the close tag.

🏃**Try it.** Now when you long-press on a reaction, you should see a particle explosion effect while it's being charged up to become a super reaction! Feel free to play with the `ReactionParticles` props until you achieve something that feels good.

## See the solution

[Solution PR](https://github.com/keith-kurak/expo-router-codemash-2025-starter/pull/1)

## Next exercise

[Module 02](02-api-routes-and-auth.md)

## Bonus

- Deploy this with EAS Update to run without a dev server (TBD)
- Make user ID a URL parameter, so workspaces can be switched via URL
