---
layout: post
title:      "Breaking Down Drag N' Drop"
date:       2020-08-30 21:41:41 -0400
permalink:  breaking_down_drag_n_drop
---


At the completion of my Javascript project, I recall being most confused while trying to read documentation on drag and drop and having a very difficult time understanding it. I hope that breaking some things down will help some of the future coders, and I hope to leave this here as a note to self for when I need the reference again.

My project is a set list maker for musicians, where you can host a list of songs that your band uses over and over again, and quickly add, edit, delete and drag and drop them into set lists that you want. You can access the project repo here: [ https://github.com/kelseyshiba/setlist_creator.](http://)

You can access the official MDN documentation here: https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API

**One-- Make items draggable**

In order to make my items, which in this case were songs - I had to give them the ability to be dragged.  This was easy!  I grabbed the element upon creation, and set the draggable attribute to true.

```
songDiv.draggable = 'true';
```


**Two-- Add Event Listeners to the draggable item**

I added the event listeners to the newly created item. In my case, I added the listeners and called another function on the song, using .this.

```
songDiv.addEventListener('dragstart', this.dragstart_handler)
songDiv.addEventListener('dragend', this.dragend_handler)
```

This meant that every song created could be dragged. 

**Three -- create functions for the listeners**

Here is the code for the functions called above:

```
dragstart_handler(e){
        e.dataTransfer.setData('text/plain', e.target.id)//e.target.id)
        //e.dataTransfer.dropEffect = 'move';
}

dragend_handler(e){   }
```

I did not end up needing to execute anything upon the END OF THE DRAG or when my user lets go of the mouse, so I could remove that, but I'm leaving it for demostration purposes.

I did need the drag start to transfer the object data I needed to attach that exact song div to the setlist, however.  This line of code is important: 

```
e.dataTransfer.setData('text/plain', e.target.id)
```

I'm taking the event dataTransfer object, and I'm telling it / setting the data that I want to send over to the drop object (my setlist). I'm only sending the ID of the song(e.target.id) and the format in which I'm sending it is in TEXT/PLAIN. Using a console log, this is the data sent:

song-1

Here's a glimpse at what the dataTransfer object looks like if you console.log it (this is just a snippet):
```
DataTransfer {dropEffect: "none", effectAllowed: "all", items: DataTransferItemList, types: Array(1), files: FileList}
dropEffect: "none"
effectAllowed: "all"
files: FileList {length: 0}
items: DataTransferItemList {0: DataTransferItem, length: 1}
types: ["text/plain"]
__proto__: DataTransfer
```

This is accessed through e.dataTransfer.

**Four -- the drop object**

Since you don't need to make the element DROPPABLE, you just need to add some event listeners to the object for the different Drop Events that are possible. Here are the four I added:

```
setlistDiv.addEventListener('dragover', this.dragover_handler)
setlistDiv.addEventListener('dragenter', this.dragenter_handler)
setlistDiv.addEventListener('dragleave', this.dragleave_handler)
setlistDiv.addEventListener('drop', this.drop_handler)
```

Here are the four corresponding functions called about on the THIS object.


```
dragover_handler(e){
        e.preventDefault();
        e.dataTransfer.dropEffect = 'move';
 }

```

You'll want to include e.preventDefault to allow the drop, and then you're allowing dataTransfer to get information with 'move'.  'Copy' is also a command used often. Otherwise, the data dropEffect is set to 'null', and you cannot get the information you need upon the drop action.

```
dragleave_handler(e){
        this.style.backgroundColor = 'rgba(0, 0, 0, 0)';
}
```

I didn't need much during this event.  However, when my user "leaves the drag" or exits the drop area, this changed the color of the area where it was back to white.

```
 dragenter_handler(e){
        e.preventDefault()
       this.style.backgroundColor = 'rgba(0, 0, 0, 0.2)';
  }
```

When entering the drop area, or dragging into it - this changes the background to be a little bit darker so the user knows where they are allowed to drag and what will accept it.

```
drop_handler(e){
        e.preventDefault( )
        const song_id = e.dataTransfer.getData('text/plain');    //song-1
        e.currentTarget.appendChild(document.getElementById(song_id))        //attach it
        this.style.backgroundColor = 'rgba(0, 0, 0, 0)';      //sets background back to normal

        setlistsongsAdapter.createSetlistSong(e) // call to fetch on the join table
    }
```

The most important piece is handling the drop.  Preventing the default, finding the thing that was attached - this was the data we sent earlier through setData, now we are using getData to access the information of the song ID, which is song-1. Then we are catching the currentTarget of the event, which is the div that encloses that larger set list div and appending the song to that.  I am then setting the background back to white after the drop is complete.  

To deal with the join table relationship, when I drag a song over to a setlist, I wanted to create a relationship between that song and that setlist, hence the final call to the method in the join table to a function that will send a fetch request and make the necessary changes on the database side. 
 
I also needed songs to be able to be removed from set lists, so I needed to deal with destroying the relationship between them and appending the song back to the songs list. 

So I added all the event listeners:

```
allSongs.addEventListener('dragover', this.dragover_handler)
allSongs.addEventListener('dragenter', this.dragenter_handler)
allSongs.addEventListener('dragleave', this.dragleave_handler)
allSongs.addEventListener('drop', this.drop_handler)
```


Created the corresponding methods:

```
static dragover_handler(e){
        e.preventDefault();
 }

    static dragleave_handler(e){
        this.style.backgroundColor = 'rgba(0, 0, 0, 0)';
    }

    static drop_handler(e){
        e.preventDefault()
        const data = e.dataTransfer.getData('text/plain');
        e.currentTarget.appendChild(document.getElementById(data))//song-id (8)
        this.style.backgroundColor = 'rgba(0, 0, 0, 0)';
        setlistsongsAdapter.deleteSetlistSong(e)
    }

    static dragenter_handler(e){
        e.preventDefault()
        this.style.backgroundColor = 'rgba(0, 0, 0, 0.2)';
    }
```

Some of these are the same as above, but again the most important piece here is the drop handler. It allows access to the song id that is being moved through getData, adds the song to the currentTarget, which is now the full list of songs, and resets the background to white. Last, it calls the delete method on the join adapter to fetch and destroy the relationship in the database.

Everything looks very initimidating when reading or learning it for the first time. I hope this makes sense for those of you out there eager to complete a new drag and drop application.  Code on! ~Kelsey


