## How to implement complex forms in Angular using FormArray

Angular is a great framework, there is no doubt in that. There is pretty much everything you would need when building a web application. One of the main things in applications like CRM, SAAS applications are user inputs.

Angular comes with a very powerful and fantastic forms module which can help in making super cool forms with validations are much more. All of us who have used Angular will have used the Forms Module for one or the other use.

## Angular Forms Modules

As I've already mentioned, the Forms module in Angular is really awesome and servers most of the purposes. There can be a difference in opinion about Angular forms especially if the form is very complex.

Complex forms will always be painful!

But If you really know how to make use of the Angular forms, most of the cases can be tackled using the in-built Angular Forms.

There are basically two types of forms Angular provides:

- Template Driven Forms
- Reactive Forms

There are tons of articles and resources on the type of forms Angular provides. The Angular docs are a great resource as well. I'm not gonna get into the roots of the type of forms Angular has to offer, rather concentrate on what we are here for.

### Angular Reactive Forms

Angular Reactive forms are great! If you haven't used it before, you should. It has a lot of awesome features that you won't get if you are using Template driven forms.

> Reactive forms use an explicit and immutable approach to managing the state of a form at a given point in time.
One of the major benefits of using Reactive forms is you can create complex forms without breaking a sweat using Reactive Forms. It is easier to design the form modals and deal with the data going in and out of the forms.

Here is how you would create a simple reactive form:

```ts
const userForm: FormGroup =  new FormGroup({
    firstName: new FormControl(''),
    lastName: new FormControl(''),
    email: new FormControl('')
  });
```

### How to Create Reactive Forms in Angular

If you directly want to jump into the topic of the post, feel free to skip this section.
- Import the Reactive Forms Module in your module.

```ts
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [
    ReactiveFormsModule
  ],
})
export class AppModule { }
```
- Create a Reactive Form using Form Builder

You can create a reactive form without using the Form Builder like in the code snippet seen above. But form builder can be really helpful in grouping form fields inside your form. And we will be needing it while dealing with Form Arrays.

```ts
import { Component, OnInit } from "@angular/core";
import { FormBuilder, FormGroup, Validators } from "@angular/forms";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.css"]
})
export class AppComponent implements OnInit {
  userForm: FormGroup;
  constructor(private formBuilder: FormBuilder) {}
  ngOnInit() {
    this.initForm();
  }

  private initForm() {
    this.userForm = this.formBuilder.group({
      firstName: ["", Validators.required],
      lastName: [],
      email: ["", [Validators.required, Validators.email]]
    });
  }
}
```

Let's break down this code:
- Import the required modules from  `@angular/forms`
- Create a `userForm` variable with a type FormGroup
- In the `ngOnit()` method, we initialize our form (I like to move the form initialize part to a different method, just to make the code a bit cleaner)
- Inject the `FormBuilder` dependency into the constructor
- Create the `formGroup` as shown above

## Angular Reactive Form Array

I was always afraid of Angular Form Arrays until I actually started using it. When I started with Angular, Reactive Forms, Form Groups, and Form Arrays were strangers and I always tried to ignore them in favor of Template driven forms. I used to use a lot of ngModels.

Form Arrays provides us a way to dynamically manage fields, meaning we can add to remove fields on the fly. Form Groups and Form Arrays are just ways to manage fields.

>FormArray is an alternative to FormGroup for managing any number of unnamed controls.

### Creating a Simple Form Array

We will start with a very simple form array and then move onto complex nested form arrays and groups.

As specified, Form Arrays are used for managing number of unnamed controls. When we need a list of items, but don't care about the control names, we can use Form Arrays. You'll get some clarity when you see the code below:
```ts
private initForm() {
    this.playlistForm = this.formBuilder.group({
      name: ["", Validators.required],
      songs: this.formBuilder.array([this.formBuilder.control("")])
    });
  }
```
Here in the above form, you can see a songs field, which is a form array containing only a single control. We use the form builder to create an array of controls. The value of the form when filled will be something like this:

```json
{
  name: "Favorites",
  songs: ["Shape of You"]
}
```

#### Adding and Removing entries from Form Array

Now that we have setup our first Form Array, let's see how it is different from Form Group and how can we make dynamic forms using form array.

Scenario: We will take a form where the user inputs his Playlist Name and Set of songs. Users can add or remove multiple songs to the songs array.

#### Add entries into Form Array

Setup the form. We can reuse the form we just created above.
Add functions for adding new fields inside the songs array.

```ts
addSong() {
  this.songs.push(this.formBuilder.control(''));
}

//Getter for songs controls
get songs() {
    return this.userForm.get("songs") as FormArray;
}
```
Now we will have two inputs for the songs form array and once we enter some value into the form, we will have the form value outputted as:

```json
{
  name: "Favorites",
  songs: ["Shape of You","Blinding Lights"]
}
```
You can keep on adding new form controls into the array by calling the `addSongs()` method. Ideally, this will be connected to an Add button on the UI which will allow the user to input more values if needed.

#### Remove entry from Form Array

Now that we know how to add items to Form Array, let see how we can delete items from the form array.

This is how we can remove entries from the form array. We basically have to remove an item off from the song's controls array. We can use the `removeAt()` property on the FormArray to remove items from the array.

```ts
removeSong(index: number) {
    this.songs.removeAt(index);
}
  
//Getter for songs controls
get songs() {
    return this.userForm.get("songs") as FormArray;
}
```
Now let's see the full code:
```ts
import { Component, OnInit } from "@angular/core";
import { FormBuilder, FormGroup, Validators, FormArray } from "@angular/forms";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.css"]
})
export class AppComponent implements OnInit {
  playlistForm: FormGroup;
  constructor(private formBuilder: FormBuilder) {}

  ngOnInit() {
    this.initForm();
  }

  /**
   * Getter for songs item as FormArray
   */
  get songs() {
    return this.playlistForm.get("songs") as FormArray;
  }
  /**
   * Add a song item to existing form array
   */
  addSong() {
    this.songs.push(this.formBuilder.control(""));
  }

  /**
   * Remove a songs item from the form array
   * @param index - index of the song item to be removed
   */
  removeSong(index: number) {
    this.songs.removeAt(index);
  }

  /**
   * Initialize the form
   */
  private initForm() {
    this.playlistForm = this.formBuilder.group({
      name: ["", Validators.required],
      songs: this.formBuilder.array([this.formBuilder.control("")])
    });
  }
}
```
```html
<form [formGroup]="userForm">
  <label>Name</label>
  <input type="text" fromControlName="name" />
  <!-- Songs Form Array -->
  <div formArrayName="songs">
    <div *ngFor="let song of songs.controls; let i=index">
      <label> Song: </label>
      <input type="text" [formControlName]="i" />
      <button (click)="addSong()">Add</button>
      <button (click)="removeSong(i)">Remove</button>
    </div>
  </div>
  <button type="submit">Submit</button>
</form>
```
Let us see what are the points that are noted when writing the template html. The distinct thing that can be seen here is `formArrayName` tag that is used in the div.

There are two things to be done for the form arrays to work:

- Here we have a getter called songs() which will return the form array. We have to assign the form array to the formArrayName property.

```html
<div formArrayName="songs"></div>
```
```ts
get songs() {
  return this.playlistForm.get("songs") as FormArray;
}
```

- Now that we have set the parent, we need to take care of the children (items inside the form array). We have to loop through all the controls inside the form array to create that many children. We do it by using the `*ngFor` structural directive. Also, note that we have set the index (let i=index). We need the index for assigning the form controls and also for removing a particular entry from the form array.

```html
<div *ngFor="let song of songs.controls; let i=index"></div>
```
- Once we have created the children, we need to attach them to their respective form controls. We do that by assigning the index to the `formControlName` property.

```html
<input type="text" [formControlName]="i" />
```

### Dealing with Complex Forms

Now that we have seen how to use Angular Form Arrays, let's dive deeper into it using form complex forms. The above example is a very simple form that was used to familiarize how the Form Array functionality can be used.

There will be many cases where we will have nested form arrays which will have form groups inside them. Dealing with nested form arrays will be a bit of a task as it can cause a lot of confusion in the template file mainly.

We are gonna go through some sample scenarios and look at how to properly design and implement complex forms in Angular by making use of Form Arrays and Form Groups.

Scenario: Let's stick with our Songs Playlist form itself, but this time instead of simply adding songs to an array, we will add albums to the array. Albums will contain an array of songs inside it. We are looking at a form where we have nested form arrays. Let me visualize the data model in JSON:

```json
{
  name: "My Favorites",
  albums: [
    {
      name: "Halcyon Days",
      artist: "Ellie Goulding",
      songs: [
        {
          name: "Burn"
        },
        {
          name: "Only You"
        },
        {
          name: "Explosions"
        }
      ]
    }
  ]
}
```

#### Modelling the Form with Nested Form Arrays

The first thing we would want to do is to identify the files and model it out in the controller. This is how the form should look like:

```ts
private initForm() {
    this.playlistForm = this.formBuilder.group({
      name: ["", Validators.required],
      albums: this.formBuilder.array([this.getAlbumItem()])
    });
  }

  private getAlbumItem() {
    return this.formBuilder.group({
      name: [],
      artist: [],
      songs: this.formBuilder.array([this.getSongItem()])
    });
  }

  private getSongItem() {
    return this.formBuilder.group({
      name: []
    });
  }
```
Here you can see that there are two fields inside the *Playlist Form*:

1. `name` - Playlist Name
2. `albums` - Albums to be made part of the playlist

The albums field is an array of *Album Item* which contains:
1. `name` - Album Name
2. `artist` - Album Artist
3. `songs` - Songs in the Album

Here the songs field is an array of *Song Item* which contains:
1. `name` - Song name

As you can see we have an albums Form Array which contains another Form Array called songs. Both the arrays are containing multiple Form Groups.

Here is how the finished controller would look like:
```ts
import { Component, OnInit } from "@angular/core";
import { FormBuilder, FormGroup, Validators, FormArray } from "@angular/forms";

@Component({
  selector: "app-playlist-album",
  templateUrl: "./playlist-album.component.html",
  styleUrls: ["./playlist-album.component.css"]
})
export class PlaylistAlbumComponent implements OnInit {
  playlistForm: FormGroup;
  constructor(private formBuilder: FormBuilder) {}

  ngOnInit() {
    this.initForm();
  }

  /**
   * Getter for album item as FormArray
   */
  get albums() {
    return this.playlistForm.get("albums") as FormArray;
  }

  /**
   * Get songs of a particular index as FormArray
   * @param albumIndex - index of the album
   */
  getSongsFormArray(albumIndex: number) {
    return this.albums.controls[albumIndex].get("songs") as FormArray;
  }

  /**
   * Get Form Controls of the songs array
   * @param albumIndex - index of the album
   */
  getSongControls(albumIndex: number) {
    return this.getSongsFormArray(albumIndex).controls;
  }

  /**
   * Add a song item to existing form array
   */
  addAlbum() {
    this.albums.push(this.getAlbumItem());
  }

  /**
   * Remove a albums item from the form array
   * @param index - index of the song item to be removed
   */
  removeAlbum(index: number) {
    this.albums.removeAt(index);
  }

  /**
   * Add song to the selected album
   * @param albumIndex - index of the album selected
   */
  addSong(albumIndex: number) {
    this.getSongsFormArray(albumIndex).push(this.getSongItem());
  }

  /**
   * Remove a song from the album
   * @param albumIndex - index of the selected album
   * @param songIndex - index of song to remove
   */
  removeSong(albumIndex: number, songIndex: number) {
    this.getSongsFormArray(albumIndex).removeAt(songIndex);
  }

  /**
   * Initialize the form
   */
  private initForm() {
    this.playlistForm = this.formBuilder.group({
      name: ["", Validators.required],
      albums: this.formBuilder.array([this.getAlbumItem()])
    });
  }

  /**
   * Create a form group for Album
   */
  private getAlbumItem() {
    return this.formBuilder.group({
      name: [],
      artist: [],
      songs: this.formBuilder.array([this.getSongItem()])
    });
  }

  /**
   * Create a form group for Song
   */
  private getSongItem() {
    return this.formBuilder.group({
      name: []
    });
  }
}
```
Let us now breakdown the code:

Firstly the parent form here is the album's form array. So we write a getter for getting the FormArray of albums:
```ts
  /**
   * Getter for albums item as FormArray
   */
  get albums() {
    return this.playlistForm.get("albums") as FormArray;
  }
```
Second, we define method to get the songs form array. This is not directly possible as each songs form array is inside the albums array. So we need the album index for getting the songs form array for that particular album.
```ts
/**
   * Get songs of a particular index as FormArray
   * @param albumIndex - index of the album
   */
  getSongsFormArray(albumIndex: number) {
    return this.albums.controls[albumIndex].get("songs") as FormArray;
  }
```

We also write a method to extract the song's form array controls so that we can iterate over it in the template. This method is not needed, we can directly call the getSongsFormArray().controls to get the controls.
```ts
 /**
   * Get Form Controls of the songs array
   * @param albumIndex - index of the album
   */
  getSongControls(albumIndex: number) {
    return this.getSongsFormArray(albumIndex).controls;
  }
```
The album's form array contains a form group that has a name, artist, and songs. We can write a method for returning us that Form Group.
```ts
/**
   * Create a form group for Album
   */
  private getAlbumItem() {
    return this.formBuilder.group({
      name: [],
      artist: [],
      songs: this.formBuilder.array([this.getSongItem()])
    });
  }
```
The song field inside the album is another form array that contains from the group. So we also write a method to get us a song item form group

```ts
/**
   * Create a form group for Song
   */
  private getSongItem() {
    return this.formBuilder.group({
      name: []
    });
  }
```
Next Up, we write methods for Adding and Removing albums. For adding an album, we just have to get hold of the album's form array and push a new control into it. You can see that in the push operation we are calling our `getAlbumItem()` method which is returning a Form Group.

For removing an Album Item, we have to grab the index of the control which needs to be removed. The template should pass the index parameter to the function and we can just remove the item off the form array.
```ts
/**
   * Add a song item to existing form array
   */
  addAlbum() {
    this.albums.push(this.getAlbumItem());
  }

  /**
   * Remove a albums item from the form array
   * @param index - index of the song item to be removed
   */
  removeAlbum(index: number) {
    this.albums..removeAt(index);
  }
```
Next, we will look at how to Add or Remove song items, we can write methods for adding a new song item and also one method for removing a particular song item. For adding a song item, we first need to specify to which album we are adding a song. We do that by providing the album index while adding the song.

While removing a song item, we have to specify which song we are removing and from which album we are removing it. This means we need to pass two indexes to the remove method. One would be the album index and the other is the songs index.
```ts
 /**
   * Add song to the selected album
   * @param albumIndex - index of the album selected
   */
  addSong(albumIndex: number) {
    this.getSongsFormArray(albumIndex).push(this.getSongItem());
  }

  /**
   * Remove a song from the album
   * @param albumIndex - index of the selected album
   * @param songIndex - index of song to remove
   */
  removeSong(albumIndex: number, songIndex: number) {
    this.getSongsFormArray(albumIndex).removeAt(songIndex);
  }
```
We have just covered all the methods that we need when we are dealing with one level of nested form arrays.

### Setting up the Template for nested Form Arrays
The hardest part is setting up the html for our form. This is hard because, the html can be a bit confusing. But once you understand the logic of writing the template to suit the form model, it is just cake walk.

I will try to make it as simple as I can. I have struggled during my initial stages when I started with Reactive Forms and Form Arrays in Angular. I also know how a beginner would see it when they first venture into the unknown grounds.

Let's get started on building the template. I am not gonna make the HTML flashy, just gonna keep things real and simple. I am also adding some styles so that it would be easy to distinguish the form arrays:

```html
  <form [formGroup]="playlistForm" class="playlist-form">
      <mat-card class="playlist-form__card">
        <mat-form-field appearance="fill">
          <mat-label>Playlist Name</mat-label>
          <input matInput formControlName="name">
        </mat-form-field>
        <div formArrayName="albums" class="albums">
          <!-- Albums Form Array ----------------------------------->
          <fieldset *ngFor="let album of albums.controls; let i=index" class="albums__item" [formGroupName]="i">
            <mat-form-field appearance="fill">
              <mat-label>Album Name</mat-label>
              <input matInput formControlName="name">
            </mat-form-field>
            <mat-form-field appearance="fill">
              <mat-label>Artist Name</mat-label>
              <input matInput formControlName="artist">
            </mat-form-field>
            <!-- Songs Form Array ----------------------------------->
            <div class="songs" formArrayName="songs">
              <fieldset class="songs__item" *ngFor="let song of getSongControls(i);let j=index" [formGroupName]="j">
                <mat-form-field appearance="fill">
                  <mat-label>Song Name</mat-label>
                  <input matInput formControlName="name">
                  <button matSuffix mat-icon-button class="song-remove-btn" (click)="removeSong(i,j)" color="warn">
                    <mat-icon>delete</mat-icon>
                  </button>
                </mat-form-field>
              </fieldset>
              <button mat-stroked-button (click)="addSong(i)" color="primary">
                <mat-icon>add</mat-icon>
              </button>
            </div>
            <!-- Songs Form Array End-------------------------------->
            <button mat-icon-button class="albums__remove" (click)="removeAlbum(i)" color="warn">
              <mat-icon>delete</mat-icon>
            </button>
          </fieldset>
          <!-- Albums Form Array End -------------------------------->
          <button mat-stroked-button (click)="addAlbum()" color="primary">
            <mat-icon>add</mat-icon>
          </button>
        </div>
        <button mat-flat-button type="submit" class="submit-btn" color="primary">Submit</button>
      </mat-card>
    </form>
```
Let's break the code down!

Firstly we have two form arrays

- Albums Form Array (Parent)
- Songs Form Array (Child)

Both of these form arrays can be spotted by following the fieldset tag in the template. The first fieldset is the albums array and the inner fieldset is for the songs array.

- Add `[formGroup]` to the main form

```html
<form [formGroup]="playlistForm"></form>
```
- Create a div for the parent form array and add the `formArryaName` property

```html
<div formArrayName="albums"></div>
```
- Add another section which we will loop through and attach the index of the loop item to `[formGroupName]` using data binding. The items inside our form array are form groups, so we need `formGroupName` to tell angular that fields inside the section are part of that particular form group.

```html
<fieldset *ngFor="let album of albums.controls; let i=index"
          [formGroupName]="i">
</fieldset>
```
- Now we have to nest the song's form array inside the album's form group. The first thing you have to do to get this done in the first approach is to simply ignore that there is already a form array. Just follow the same as what you did in steps 2 & 3.

```html
<div formArrayName="songs"></div>
```
- Now we create a section for the songs form group which loop through the number of controls present in the songs array.

```html
<fieldset *ngFor="let song of getSongControls(i);let j=index"
          formGroupName]="j">
<fieldset>
```

We are done! Now if you go back and see the steps, It is exactly the same except we have changes the array names and the list of controls to be looped.

Form Array becomes very complex because of the nested HTML elements. The best way to get over it is to individually develop the form groups and then put the child forms inside the parent. Or just follow a pattern and add some comments so that you don't get confused.

It is very simple and straightforward as you see here!

## Interactive Demo

![form-array-demo.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1607786988443/T2o4Gy8_S.png)

I've created a simple demo, which will help you see how the form is being modified when you add or remove entries to the form array.

**Demo**: https://brave-payne-95d429.netlify.com/

**Source Code**: https://github.com/adisreyaj/angular-form-array-demo

## 🌎 Connect with me

- [Twitter](https://twitter.com/AdiSreyaj)
- [Github](https://github.com/adisreyaj)
- [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
- [Cartella](https://cartella.sreyaj.dev) - building at the moment

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1618661389599/2B667-okT.png)](https://www.buymeacoffee.com/adisreyaj)

Do add your thoughts in the comments section.
Stay Safe ❤️
