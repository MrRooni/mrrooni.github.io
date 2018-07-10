---
author: MrRooni
comments: false
date: 2013-02-19 18:56:05+00:00
layout: post
slug: uitableview-and-nsfetchedresultscontroller-updates-done-right
title: 'UITableView and NSFetchedResultsController: Updates Done Right'
wordpress_id: 635
categories:
- Cocoa
- Sample Code
---

While working on [MoneyWell Express](http://nothirst.com/moneywellexpress/) 1.0 I decided to finally sit down and figure out a bug that had plagued me for a long time: Periodic and seemingly random crashes when updating MoneyWell's transaction UITableView. If you've spent any significant time with UITableView you've undoubtably seen an error similar to this one:


    *** Assertion failure in -[UITableView _endCellAnimationsWithContext:], /SourceCache/UIKit_Sim/UIKit-2380.17/UITableView.m:1070
    CoreData: error: Serious application error.
    An exception was caught from the delegate of NSFetchedResultsController during a call to -controllerDidChangeContent:.
    Invalid update: invalid number of rows in section 2.
    The number of rows contained in an existing section after the update (2) must be equal to the number of rows contained in that section before the update (1),
    plus or minus the number of rows inserted or deleted from that section (0 inserted, 0 deleted) and
    plus or minus the number of rows moved into or out of that section (0 moved in, 0 moved out). with userInfo (null)


The problem with this bug for me was that it was intermittent and never reliably reproducible - until it was. One day while working on our [syncing framework](https://github.com/nothirst/TICoreDataSync/) I had this issue start to reproduce itself every time MoneyWell Express attempted to consume some sync changes and I seized the opportunity to finally figure out what was going on.

Let's start by taking a look at the sample code Apple provides on the [NSFetchedResultsControllerDelegate Protocol Reference](http://developer.apple.com/library/ios/#documentation/CoreData/Reference/NSFetchedResultsControllerDelegate_Protocol/Reference/Reference.html):


```objective-c
    /*
       Assume self has a property 'tableView' -- as is the case for an instance of a UITableViewController
       subclass -- and a method configureCell:atIndexPath: which updates the contents of a given cell
       with information from a managed object at the given index path in the fetched results controller.
     */

    - (void)controllerWillChangeContent:(NSFetchedResultsController *)controller
    {
        [self.tableView beginUpdates];
    }

    - (void)controller:(NSFetchedResultsController *)controller didChangeSection:(id )sectionInfo
        atIndex:(NSUInteger)sectionIndex forChangeType:(NSFetchedResultsChangeType)type
    {
        switch (type) {
            case NSFetchedResultsChangeInsert:
                [self.tableView insertSections:[NSIndexSet indexSetWithIndex:sectionIndex] withRowAnimation:UITableViewRowAnimationFade];
                break;
            case NSFetchedResultsChangeDelete:
                [self.tableView deleteSections:[NSIndexSet indexSetWithIndex:sectionIndex] withRowAnimation:UITableViewRowAnimationFade];
                break;
        }
    }

    - (void)controller:(NSFetchedResultsController *)controller didChangeObject:(id)anObject atIndexPath:(NSIndexPath *)indexPath
              forChangeType:(NSFetchedResultsChangeType)type newIndexPath:(NSIndexPath *)newIndexPath
    {
        UITableView *tableView = self.tableView;

        switch (type) {
            case NSFetchedResultsChangeInsert:
                [tableView insertRowsAtIndexPaths:[NSArray arrayWithObject:newIndexPath] withRowAnimation:UITableViewRowAnimationFade];
                break;
            case NSFetchedResultsChangeDelete:
                [tableView deleteRowsAtIndexPaths:[NSArray arrayWithObject:indexPath] withRowAnimation:UITableViewRowAnimationFade];
                break;

            case NSFetchedResultsChangeUpdate:
                [self configureCell:[tableView cellForRowAtIndexPath:indexPath] atIndexPath:indexPath];
                break;

            case NSFetchedResultsChangeMove:
                [tableView deleteRowsAtIndexPaths:[NSArray arrayWithObject:indexPath] withRowAnimation:UITableViewRowAnimationFade];
                [tableView insertRowsAtIndexPaths:[NSArray arrayWithObject:newIndexPath] withRowAnimation:UITableViewRowAnimationFade];
                break;
        }
    }

    - (void)controllerDidChangeContent:(NSFetchedResultsController *)controller
    {
        [self.tableView endUpdates];
    }
```

As you can see this code starts the tableView updates in controllerWillChangeContent:, responds to each change as it happens, and then ends the tableView updates in controllerDidChangeContent:. The problem I ran into with this code is that inserting sections into the table also inserted all the rows for that new section, but since those rows were also being reported as inserted we would get twice the number of rows inserted when adding a new section to the table. The answer was to queue up all the updates that the fetchedResultsController reported and then respond to them all at once, like so:


```objective-c
    @interface SomeViewController ()

    // Declare some collection properties to hold the various updates we might get from the NSFetchedResultsControllerDelegate
    @property (nonatomic, strong) NSMutableIndexSet *deletedSectionIndexes;
    @property (nonatomic, strong) NSMutableIndexSet *insertedSectionIndexes;
    @property (nonatomic, strong) NSMutableArray *deletedRowIndexPaths;
    @property (nonatomic, strong) NSMutableArray *insertedRowIndexPaths;
    @property (nonatomic, strong) NSMutableArray *updatedRowIndexPaths;

    @end

    @implementation SomeViewController

    #pragma mark - NSFetchedResultsControllerDelegate methods

    - (void)controller:(NSFetchedResultsController *)controller didChangeObject:(id)anObject atIndexPath:(NSIndexPath *)indexPath
              forChangeType:(NSFetchedResultsChangeType)type newIndexPath:(NSIndexPath *)newIndexPath
    {
        if (type == NSFetchedResultsChangeInsert) {
            if ([self.insertedSectionIndexes containsIndex:newIndexPath.section]) {
                // If we've already been told that we're adding a section for this inserted row we skip it since it will handled by the section insertion.
                return;
            }

            [self.insertedRowIndexPaths addObject:newIndexPath];
        } else if (type == NSFetchedResultsChangeDelete) {
            if ([self.deletedSectionIndexes containsIndex:indexPath.section]) {
                // If we've already been told that we're deleting a section for this deleted row we skip it since it will handled by the section deletion.
                return;
            }

            [self.deletedRowIndexPaths addObject:indexPath];
        } else if (type == NSFetchedResultsChangeMove) {
            if ([self.insertedSectionIndexes containsIndex:newIndexPath.section] == NO) {
                [self.insertedRowIndexPaths addObject:newIndexPath];
            }

            if ([self.deletedSectionIndexes containsIndex:indexPath.section] == NO) {
                [self.deletedRowIndexPaths addObject:indexPath];
            }
        } else if (type == NSFetchedResultsChangeUpdate) {
            [self.updatedRowIndexPaths addObject:indexPath];
        }
    }

    - (void)controller:(NSFetchedResultsController *)controller didChangeSection:(id )sectionInfo atIndex:(NSUInteger)sectionIndex
              forChangeType:(NSFetchedResultsChangeType)type
    {
        switch (type) {
            case NSFetchedResultsChangeInsert:
                [self.insertedSectionIndexes addIndex:sectionIndex];
                break;
            case NSFetchedResultsChangeDelete:
                [self.deletedSectionIndexes addIndex:sectionIndex];
                break;
            default:
                ; // Shouldn't have a default
                break;
        }
    }

    - (void)controllerDidChangeContent:(NSFetchedResultsController *)controller
    {
        [self.tableView beginUpdates];

        [self.tableView deleteSections:self.deletedSectionIndexes withRowAnimation:UITableViewRowAnimationAutomatic];
        [self.tableView insertSections:self.insertedSectionIndexes withRowAnimation:UITableViewRowAnimationAutomatic];

        [self.tableView deleteRowsAtIndexPaths:self.deletedRowIndexPaths withRowAnimation:UITableViewRowAnimationLeft];
        [self.tableView insertRowsAtIndexPaths:self.insertedRowIndexPaths withRowAnimation:UITableViewRowAnimationRight];
        [self.tableView reloadRowsAtIndexPaths:self.updatedRowIndexPaths withRowAnimation:UITableViewRowAnimationAutomatic];

        [self.tableView endUpdates];

        // nil out the collections so they are ready for their next use.
        self.insertedSectionIndexes = nil;
        self.deletedSectionIndexes = nil;
        self.deletedRowIndexPaths = nil;
        self.insertedRowIndexPaths = nil;
        self.updatedRowIndexPaths = nil;
    }

    #pragma mark - Overridden getters

    /**
     * Lazily instantiate these collections.
     */

    - (NSMutableIndexSet *)deletedSectionIndexes
    {
        if (_deletedSectionIndexes == nil) {
            _deletedSectionIndexes = [[NSMutableIndexSet alloc] init];
        }

        return _deletedSectionIndexes;
    }

    - (NSMutableIndexSet *)insertedSectionIndexes
    {
        if (_insertedSectionIndexes == nil) {
            _insertedSectionIndexes = [[NSMutableIndexSet alloc] init];
        }

        return _insertedSectionIndexes;
    }

    - (NSMutableArray *)deletedRowIndexPaths
    {
        if (_deletedRowIndexPaths == nil) {
            _deletedRowIndexPaths = [[NSMutableArray alloc] init];
        }

        return _deletedRowIndexPaths;
    }

    - (NSMutableArray *)insertedRowIndexPaths
    {
        if (_insertedRowIndexPaths == nil) {
            _insertedRowIndexPaths = [[NSMutableArray alloc] init];
        }

        return _insertedRowIndexPaths;
    }

    - (NSMutableArray *)updatedRowIndexPaths
    {
        if (_updatedRowIndexPaths == nil) {
            _updatedRowIndexPaths = [[NSMutableArray alloc] init];
        }

        return _updatedRowIndexPaths;
    }

    @end
```

This implementation properly queues all the changes, makes sure not to insert or delete any rows when they are part of an inserted or deleted section, and updates the table in one nice little chunk. You don't need to worry about implementing the willChangeContent: delegate method. It also has the benefit that, if you were so inclined, you could see how many updates you were about to perform on the tableView and just call reloadData instead, like so:


```objective-c
    - (void)controllerDidChangeContent:(NSFetchedResultsController *)controller
    {
        NSInteger totalChanges = [self.deletedSectionIndexes count] +
                                 [self.insertedSectionIndexes count] +
                                 [self.deletedRowIndexPaths count] +
                                 [self.insertedRowIndexPaths count] +
                                 [self.updatedRowIndexPaths count];
        if (totalChanges > 50) {
            self.insertedSectionIndexes = nil;
            self.deletedSectionIndexes = nil;
            self.deletedRowIndexPaths = nil;
            self.insertedRowIndexPaths = nil;
            self.updatedRowIndexPaths = nil;

            [self.tableView reloadData];
            return;
        }

        [self.tableView beginUpdates];

        [self.tableView deleteSections:self.deletedSectionIndexes withRowAnimation:UITableViewRowAnimationAutomatic];
        [self.tableView insertSections:self.insertedSectionIndexes withRowAnimation:UITableViewRowAnimationAutomatic];

        [self.tableView deleteRowsAtIndexPaths:self.deletedRowIndexPaths withRowAnimation:UITableViewRowAnimationLeft];
        [self.tableView insertRowsAtIndexPaths:self.insertedRowIndexPaths withRowAnimation:UITableViewRowAnimationRight];
        [self.tableView reloadRowsAtIndexPaths:self.updatedRowIndexPaths withRowAnimation:UITableViewRowAnimationAutomatic];

        [self.tableView endUpdates];

        self.insertedSectionIndexes = nil;
        self.deletedSectionIndexes = nil;
        self.deletedRowIndexPaths = nil;
        self.insertedRowIndexPaths = nil;
        self.updatedRowIndexPaths = nil;
    }
```

If you've got any questions or if you notice some horrible bug that I've introduced let me know on [Twitter](http://twitter.com/MrRooni) or [App.net](https://alpha.app.net/mrrooni), I'm MrRooni on both.

And if you're the kind of person that likes gists, you can find the above code on GitHub here: [https://gist.github.com/MrRooni/4988922](https://gist.github.com/MrRooni/4988922)
