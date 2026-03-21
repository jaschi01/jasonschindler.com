---
layout: post
title: "Getting started with the Revit API in C#"
date: 2026-03-21
tags: [Revit, C#, Revit API]
excerpt: "A practical introduction to building your first Revit add-in — covering the IExternalCommand interface, the all-important transaction model, and a few gotchas that will save you hours."
---

If you've ever searched for Revit API documentation and come away more confused than when you started, you're not alone. The official docs are sparse, the forums are full of outdated code, and most examples skip straight to the interesting parts without explaining the scaffolding underneath.

This post is the introduction I wish I'd had. We'll build a minimal but real Revit add-in in C# — one that actually does something useful — and I'll call out the parts that tripped me up early on.

<!--more-->

## The basic structure

Every Revit add-in implements `IExternalCommand`. That interface has exactly one method you need to care about:

```csharp
public Result Execute(
    ExternalCommandData commandData,
    ref string message,
    ElementSet elements)
```

The `commandData` parameter is your entry point to everything — the active document, the active view, the selection, and the Revit application itself.

```csharp
UIApplication uiApp = commandData.Application;
UIDocument uiDoc = uiApp.ActiveUIDocument;
Document doc = uiDoc.Document;
```

## Transactions are not optional

This is the one that bites everyone. Any time you modify the model — changing a parameter, creating an element, deleting something — you need to wrap it in a transaction:

```csharp
using (Transaction tx = new Transaction(doc, "My Change"))
{
    tx.Start();
    // ... make your changes here ...
    tx.Commit();
}
```

Skip the transaction and Revit will throw an `InvalidOperationException` at runtime with a message that doesn't make it obvious what went wrong. The `using` block ensures the transaction is rolled back cleanly if something throws.

## A minimal working example

Here's a complete add-in that reads the `Mark` parameter from every selected element and reports it in a dialog:

```csharp
[Transaction(TransactionMode.ReadOnly)]
public class ReportMarks : IExternalCommand
{
    public Result Execute(
        ExternalCommandData commandData,
        ref string message,
        ElementSet elements)
    {
        UIDocument uiDoc = commandData.Application.ActiveUIDocument;
        ICollection<ElementId> selectedIds = uiDoc.Selection.GetElementIds();

        if (!selectedIds.Any())
        {
            TaskDialog.Show("Report", "No elements selected.");
            return Result.Cancelled;
        }

        var lines = new List<string>();
        foreach (ElementId id in selectedIds)
        {
            Element el = uiDoc.Document.GetElement(id);
            Parameter mark = el.get_Parameter(BuiltInParameter.ALL_MODEL_MARK);
            string value = mark?.AsString() ?? "(no mark)";
            lines.Add($"{el.Name}: {value}");
        }

        TaskDialog.Show("Marks", string.Join("\n", lines));
        return Result.Succeeded;
    }
}
```

Notice the `[Transaction(TransactionMode.ReadOnly)]` attribute on the class — since we're only reading data here, we declare the command as read-only. Revit uses this hint to decide whether it needs to do any document bookkeeping before running your command.

## What's next

In a follow-up post I'll cover the `.addin` manifest file that tells Revit about your command, and how to set up a Visual Studio project that copies the DLL to the right place on build so you can test quickly without manual file shuffling.

If you're working on Revit add-ins and run into something confusing, feel free to reach out — there's a good chance I've hit the same wall.
