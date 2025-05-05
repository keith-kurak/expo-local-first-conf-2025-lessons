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

1. Inside **shared/schema.ts**, in the `reaction` table, add the `type` field:

```ts
type: State.SQLite.text({
  schema: Schema.Literal("regular", "super"),
  default: "regular",
}),
```

2. In the same file, look at the `materializers`. These are the mutations- the events that make up the change history in Livestore. We need to add a new materializer that will create a note with a `type`.

Notice that the materializers so far have a `v1` in front. Once a materializer is present, it cannot be deleted, or else it would not be possible to process paste events. So, it can be helpful to create a new `v2` materializer that incorporates the `type` field. Add the following to the `materializers` list:

```ts
"v2.NoteReactionCreated": ({ id, noteId, emoji, type, createdBy }) =>
  reaction.insert({
    id,
    noteId,
    emoji,
    type: type as "regular" | "super",
    createdBy,
    createdAt: new Date(),
  }),
```

Since `type` has a default value, the `v1` matieralizer will still work on a client version where it has been superceded by the `v2` materializer.

3. Let's also add the materializer for updating just the type on an existing reaction. Add the following:

```ts
"v2.NoteReactionTypeUpdated": ({ id, type }) =>
  reaction.update({ type }).where({ id }),
```

Let's update the bottom sheet that pops up when you'd like to add a reaction to accept the new type parameter. This also serves as a bit of an example of how file-based routing works in an Expo app. The bottom sheet is in **packages/mobile/app/(home)/reaction/[note].tsx**. If you look inside the **packages/mobile/app/(home)/_layout.tsx** file, you'll see the `reaction/[note]` route referenced with `presentation: "formSheet"`. This means that the bottom sheet is a separate screen that pushes on top of the list of notes, but is partially transparent, so you can still see the list underneath it. The `[note]` part of the route is where the note ID is passed, so the screen knows which note the reaction is for.

1. In **app/(home)/reaction/[note].tsx**, update `handleReaction` to incorporate the type:

```diff
- function handleReaction(emoji: string) {
+ function handleReaction(emoji: string, type: "regular" | "super") {
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
-  onPress={() => handleReaction(reaction)}
+  onPress={() => handleReaction(reaction, "regular")}
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
      deletedAt: null
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

TBD

## Exercise 4: Fancy animations!

Super reactions aren't really all that super until they have some pizzaz. We're going to add a cool particle effect using React Native Reanimated. This will occur while the long press is in progress, like you're charging up until the reaction is "super".

TBD

## See the solution

[Solution PR](https://github.com/keith-kurak/expo-router-codemash-2025-starter/pull/1)

## Next exercise

[Module 02](02-api-routes-and-auth.md)

## Bonus
- Deploy this with EAS Update to run without a dev server (TBD)
- Make user ID a URL parameter, so workspaces can be switched via URL