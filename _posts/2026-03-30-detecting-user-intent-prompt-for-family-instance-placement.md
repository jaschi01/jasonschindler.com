---
layout: post
title: "Did the User Actually Place Anything? Detecting Intent After PromptForFamilyInstancePlacement"
date: 2026-03-30
categories: [revit-api, c-sharp]
tags: [revit, api, families, events, uiDocument]
---

If you've ever built a Revit add-in that places family instances programmatically, you've almost certainly reached for `UIDocument.PromptForFamilyInstancePlacement`. It hands control over to the user, lets them click around the canvas to place one or more instances of a given `FamilySymbol`, and then returns when they're done. Clean, native, and exactly what users expect.

The trouble is: *done* is doing a lot of work in that sentence. "Done" might mean the user placed ten instances and hit Escape to exit. It might also mean they immediately hit Escape and placed nothing at all. From the method's perspective, those two outcomes are identical — it returns `void` either way, throws no exception, and gives you no signal whatsoever about what just happened.

This is one of those quiet Revit API gotchas that doesn't announce itself until you're in QA and someone asks, "What happens if the user cancels?"

---

## What `PromptForFamilyInstancePlacement` Actually Does

The method signature looks like this:

```csharp
public void PromptForFamilyInstancePlacement(FamilySymbol symbol)
```

You pass it an activated `FamilySymbol`, and Revit enters the standard family placement mode — the same cursor-and-preview experience users get when placing families manually. They can click to place instances, switch types via the Options Bar, and exit by pressing Escape or switching tools. At that point, control returns to your code.

It's genuinely useful. You don't have to manage the placement loop yourself, and users get a familiar, first-class experience rather than a custom dialog that feels bolted on.

---

## The Core Problem: Void Means Nothing

Because the method returns `void`, there's no return value to inspect. There's also no `OperationCanceledException` or similar when the user cancels — Revit treats Escape as a normal exit, not a fault condition. If you're expecting an exception to branch on, you'll be waiting a long time.

This means you cannot, in any straightforward way, answer the question: *did anything get placed?*

---

## The Standard Solution: Subscribe to `DocumentChanged`

The reliable pattern is to hook into the `Application.DocumentChanged` event *before* calling the method, collect the IDs of any elements added during the session, and then inspect that collection once the method returns.

Here's the full pattern:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Autodesk.Revit.DB;
using Autodesk.Revit.DB.Events;
using Autodesk.Revit.UI;

public void PlaceFamilyWithIntentCheck(UIDocument uiDoc, FamilySymbol symbol)
{
    var placedIds = new List<ElementId>();

    void OnDocumentChanged(object sender, DocumentChangedEventArgs e)
    {
        placedIds.AddRange(e.GetAddedElementIds());
    }

    if (!symbol.IsActive)
        symbol.Activate();

    uiDoc.Application.Application.DocumentChanged += OnDocumentChanged;

    try
    {
        uiDoc.PromptForFamilyInstancePlacement(symbol);
    }
    finally
    {
        uiDoc.Application.Application.DocumentChanged -= OnDocumentChanged;
    }

    if (placedIds.Count == 0)
    {
        // User cancelled without placing anything — handle accordingly
        TaskDialog.Show("Placement", "No instances were placed.");
        return;
    }

    // Filter to just the family instances matching our symbol
    var doc = uiDoc.Document;
    var placedInstances = placedIds
        .Select(id => doc.GetElement(id))
        .OfType<FamilyInstance>()
        .Where(fi => fi.Symbol.Id == symbol.Id)
        .ToList();

    TaskDialog.Show("Placement", $"{placedInstances.Count} instance(s) placed.");
}
```

The `try/finally` block is important — it guarantees you unsubscribe even if something goes wrong during placement. Leaving event handlers dangling is a reliable path to subtle, hard-to-reproduce bugs.

---

## Filter `GetAddedElementIds()` — Don't Trust It Blindly

`DocumentChanged` fires for *every* change during the placement session, not just your family instances. Revit may quietly add analytical nodes, tags, or other elements as side effects. If you're just checking `placedIds.Count > 0`, you might get a false positive.

The fix is to filter added IDs against the `FamilyInstance` type and cross-check the `Symbol.Id` against the symbol you passed in. This ensures you're counting only what your user intentionally placed, and nothing else.

---

## Common Mistakes to Avoid

**Don't wrap the call in a transaction.** `PromptForFamilyInstancePlacement` manages its own transactions internally. Wrapping it in an outer `Transaction` will throw an exception because Revit won't allow a placement operation inside an already-open transaction.

**Don't expect an exception on Escape.** It won't come. If your post-placement logic is gated on catching a cancellation exception, it will never run.

**Don't look for a return value.** There isn't one. No overload returns a collection of placed IDs, a status code, or anything else useful. The event subscription pattern is the intended approach.

---

## Wrapping Up

`PromptForFamilyInstancePlacement` is a convenient method that solves a real UX problem, but its silent cancellation behaviour catches developers off guard more often than it should. The `DocumentChanged` subscription pattern isn't glamorous, but it's solid, well-understood, and gives you exactly the information you need. Subscribe before, unsubscribe after (in a `finally`), filter the results, and you'll have a reliable picture of what the user actually did.

Once you've seen this gotcha once, it sticks — but hopefully this post saves you from having to discover it the hard way in production.
