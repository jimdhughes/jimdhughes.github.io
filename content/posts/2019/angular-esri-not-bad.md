+++
author = "James D Hughes"
date = 2019-09-06T09:33:18Z
description = ""
draft = false
slug = "angular-esri-not-bad"
title = "Angular + ESRI = Not bad"

+++


There's no question that I'm an ng fanboy.

React's willy nilly do whatever you want attitude doesn't do much for me.

Vue is basically react. Let's be serious.

I like the clear separation of my HTML, CSS, TS, and unit tests. That won't ever go away.   Work has taken me into some new territories where I didn't think I'd have as much fun as I am. That's enough of my ranting about javascript frameworks. Let's dive into the root of the article!

If you don't feel like reading below, the code is up on my github. Check it out at [https://github.com/jimdhughes/jdhc-esri-starter](https://github.com/jimdhughes/jdhc-esri-starter)

If you feel like following along with some code and some possible random side stories, follow along!

Since I value productivity over feeling clever, I'm going to use the angluar cli. I hope you have it installed too.  We're going to use Angular v8. I typically add routing and use SCSS but feel free to make it your own.

```terminal
ng new esri-app
```

I absolutely LOVE the Angular team's Material libraries as well as the flex-layout library, so those are the FIRST things that I install.  Being a proponent of functional, performant and quick to develop, I have no problem admitting that I'd prefer to use some angular directives over writing CSS. You can read the official documentation over at [https://material.angular.io/guide/getting-started](https://material.angular.io/guide/getting-started) and [https://github.com/angular/flex-layout](https://github.com/angular/flex-layout) or you can just do what I tell you below.

```terminal
cd esri-app
npm install --save @angular/material @angular/cdk @angular/animations @angular/flex-layout
```

After waiting a few minutes, you need to add some things into your `app.module.ts` file.

```typescript
// ... Some imports
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';

@NgModule({
  declarations: [],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    AppRoutingModule,
  ],
  entryComponents: [],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}

```

I have no shame in importing the entire angular material library into a helper module, especially when I don't expect the app to ever make it past my desktop or workstation.  There's something to be said about just having everything up front then optimizing when you have to. So we're going to use the stackblitz template that google provides but I'll use the CLI to make live a bit more structured.

```terminal
ng g m material
```

Now you'll have to edit your `material.module.ts` file to include all the material directives as well as the flex-layout directives.

```typescript
// src/app/material/material.module.ts

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { A11yModule } from '@angular/cdk/a11y';
import { BidiModule } from '@angular/cdk/bidi';
import { ObserversModule } from '@angular/cdk/observers';
import { OverlayModule } from '@angular/cdk/overlay';
import { PlatformModule } from '@angular/cdk/platform';
import { PortalModule } from '@angular/cdk/portal';
import { ScrollingModule } from '@angular/cdk/scrolling';
import { CdkStepperModule } from '@angular/cdk/stepper';
import { CdkTableModule } from '@angular/cdk/table';
import { CdkTreeModule } from '@angular/cdk/tree';
import {
  MatAutocompleteModule,
  MatBadgeModule,
  MatBottomSheetModule,
  MatButtonModule,
  MatButtonToggleModule,
  MatCardModule,
  MatCheckboxModule,
  MatChipsModule,
  MatDatepickerModule,
  MatDialogModule,
  MatDividerModule,
  MatExpansionModule,
  MatFormFieldModule,
  MatGridListModule,
  MatIconModule,
  MatInputModule,
  MatListModule,
  MatMenuModule,
  MatNativeDateModule,
  MatPaginatorModule,
  MatProgressBarModule,
  MatProgressSpinnerModule,
  MatRadioModule,
  MatRippleModule,
  MatSelectModule,
  MatSidenavModule,
  MatSliderModule,
  MatSlideToggleModule,
  MatSnackBarModule,
  MatSortModule,
  MatStepperModule,
  MatTableModule,
  MatTabsModule,
  MatToolbarModule,
  MatTooltipModule,
  MatTreeModule,
} from '@angular/material';

import { FlexLayoutModule } from '@angular/flex-layout';

@NgModule({
  declarations: [],
  imports: [
    CommonModule
  ],
  exports: [
    // CDK
    A11yModule,
    BidiModule,
    ObserversModule,
    OverlayModule,
    PlatformModule,
    PortalModule,
    ScrollingModule,
    CdkStepperModule,
    CdkTableModule,
    CdkTreeModule,

    // Material
    MatAutocompleteModule,
    MatBadgeModule,
    MatBottomSheetModule,
    MatButtonModule,
    MatButtonToggleModule,
    MatCardModule,
    MatCheckboxModule,
    MatChipsModule,
    MatDatepickerModule,
    MatDialogModule,
    MatDividerModule,
    MatExpansionModule,
    MatFormFieldModule,
    MatGridListModule,
    MatIconModule,
    MatInputModule,
    MatListModule,
    MatMenuModule,
    MatNativeDateModule,
    MatPaginatorModule,
    MatProgressBarModule,
    MatProgressSpinnerModule,
    MatRadioModule,
    MatRippleModule,
    MatSelectModule,
    MatSidenavModule,
    MatSliderModule,
    MatSlideToggleModule,
    MatSnackBarModule,
    MatSortModule,
    MatStepperModule,
    MatTableModule,
    MatTabsModule,
    MatToolbarModule,
    MatTooltipModule,
    MatTreeModule,
    // Flex
    FlexLayoutModule,
  ]
})
export class MaterialModule { }

/**  Copyright 2019 Google Inc. All Rights Reserved.
    Use of this source code is governed by an MIT-style license that
    can be found in the LICENSE file at http://angular.io/license */

^ That ought to pad my word count. I included the copyright because that's just good juju for this guy.

Now that we've got this delightful helper set up. Import it into your `app.module.ts` file to expose it to this module of your app!  I'm not going to copy the code block. Just go do it.

Next piece of prep is to add icons to my `index.html`...

```html
<!-- index.html -->

<head>
    <!-- ... -->
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
</head>

```

Then import a theme into `styles.scss`

```scss
@import "~@angular/material/prebuilt-themes/indigo-pink.css";
```

Sure. That looks pretty good.

Now that all that is out of the way. Make sure it works by deleting almost everything in your `app.component.html` and give it a toolbar.

```html
<mat-toolbar color="primary">
    Esri Starter
</mat-toolbar>
<router-outlet></router-outlet>
```

Start up the app with `ng serve` and check out the magic!

## This is the time I realized that the computer I was writing my blog on was running Angular CLI v7 and I installed all v8 dependencies.. so here we go upgrading..

```terminal
ng update @angular/cli @angular/core
```

Yeah. It's only that easy because we haven't done anything yet. You can read their upgrade guides over at [https://update.angular.io/#7.0:8.0](https://update.angular.io/#7.0:8.0) though.

After a short blip.. here we go again! - Your app should be running now.

Those annoying margins! I forgot about my trusty angular scss defaults!

```scss
html,
body {
  height: 100%;
  width: 100%;
  margin: 0;
}

router-outlet.router-flex + * {
  display: flex;
  flex: 1 1 auto;
  flex-direction: column;
}

```

That router-outlet stuff is a bonus for you later - if you've worked with these angular router and couldn't get the children to fill the height, you'll like it.

ALRIGHT! Enough already - let's get to the ESRI stuff!

First we need to install some more libraries.. esri-loader for importing the annoying dojo components and @types/arcgis-js-api for ... types

```terminal
npm install --save esri-loader @types/arcgis-js-api
```

Update your `tsconfig.app.json` to include the esri typings

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "outDir": "../out-tsc/app",
    "types": ["arcgis-js-api"]
  },
  "exclude": ["test.ts", "**/*.spec.ts"]
}

```

You should also add the esri styles to your index.html.  We're using 4.12 for this article. Upgrade if you need to.

```html
<head>
    <!-- ... -->
    <link rel="stylesheet" href="https://js.arcgis.com/4.12/esri/css/main.css">
</head>
```

Now to build the Arcmap component. You guessed it. Another CLI command

```terminal
ng g c arcmap
```

Then we add it into our AppComponent. Since I like maps so much, I'm going to make it the focal point of my app.  Here is some code to make a generic sidenav component.

```html
<!-- app.component.html -->
<div fxLayout="row" fxFlexFill>
  <mat-drawer-container class="example-container" fxLayout="column" fxFlex>
    <mat-drawer mode="side" [opened]="true" class="app-sidenav">
      <mat-card>
        A Card!
      </mat-card>
    </mat-drawer>
    <mat-drawer-content fxLayout="column">
      <mat-toolbar fxLayout="row" color="primary">
        <button mat-icon-button (click)="onMenuToggle()">
          <mat-icon>menu</mat-icon>
        </button>
        <h3>ESRI Starter</h3>
      </mat-toolbar>
      <div fxFlex fxLayout="row">
        <router-outlet></router-outlet>
        <app-arcmap fxFlex></app-arcmap>
      </div>
    </mat-drawer-content>
  </mat-drawer-container>
</div>

```

I added a class to `app.component.scss` to make my sidenav a little more sidenavy

```scss
.app-sidenav {
  width: 250px;
}

```

Refresh your app and you'll see a side menu with a card, and a component that just says `arcmap works!` Congrats!

Now for the fun stuff! Below is legit all you'll need to render that map.

```html
<!-- arcmap.component.html -->
<div class="mapview" #arcmap>
</div>
```

... Eventualy.

We need to do some magic in the typescript file still! Let's import things and set up our viewchild.

```typescript
import {
  Component,
  OnInit,
  ViewChild,
  ElementRef,
  HostListener
} from "@angular/core";
import { loadModules } from "esri-loader";
import esri = __esri;

@Component({
  selector: "app-arcmap",
  templateUrl: "./arcmap.component.html",
  styleUrls: ["./arcmap.component.scss"]
})
export class ArcmapComponent implements OnInit {
  @ViewChild("arcmap", { static: false })
  private arcMapRef: ElementRef;
  mapView: esri.MapView;
  map: esri.Map;

  constructor() {}

  ngOnInit() {
    
  }
}
```

Still not broken!

Now let's init the map.

```typescript
  ngOnInit() {
    this.init();
  }

  async init() {
    try {
      const [Map, MapView] = (await loadModules([
        "esri/Map",
        "esri/views/MapView"
      ])) as [esri.MapConstructor, esri.MapViewConstructor];

      this.map = new Map({
        basemap: "dark-gray"
      });
      this.mapView = new MapView({
        map: this.map,
        center: [-113.4909, 53.544],
        zoom: 12,
        container: this.arcMapRef.nativeElement,
        ui: {
          components: ["attribution"]
        }
      });
    } catch (e) {
      console.error(e);
    }
  }
```

I call another init because I like async/await and the lifecycle hooks yell at me when I throw an async in front of them. This just makes life a bit easier.

Hit save and you see ...

#NOTHING!

This is because that div is very small. There is nothing HTMLy in it so it's tiny. We can fix that with some scss in `arcmap.component.scss`

```scss
.mapview {
    height: 100%;
    width:100%;
    overflow: hidden;
}

.ng-star-inserted {
    height: 100%
}

```

Pow. You have an ESRI map sitting in your Angular app. It's centered on Edmonton because that's where I'm from.  Update the lat/lon in the init code to set it on your city if you're so interested.

Now is as good a time as any to add in a service for managing the state of our Map. To do this, I like to create a services module

`ng g m services`

`ng g s services/arcmap`

I decided that all I really want to do for my app is to set the center and zoom levels because really, what's a map that doesn't move around and zoom?

Inside of `src/services/arcmap.service.ts`...

```typescript
import { Injectable, EventEmitter, Output } from '@angular/core';

export interface Point {
  lon: number;
  lat: number;
}

@Injectable({
  providedIn: 'root'
})
export class ArcmapService {
  private mapZoom = 12;
  private mapCenter: Point = { lat: -113.4909, lon: 53.544 };

  esriCenter: Point = { lat: -113.4909, lon: 53.544 };
  esriZoom = 12;

  @Output()
  zoomEmitter: EventEmitter<number> = new EventEmitter<number>();
  @Output()
  centerEmitter: EventEmitter<Point> = new EventEmitter<Point>();

  constructor() {}

  setCenter(p: Point) {
    this.mapCenter = p;
    this.centerEmitter.next(this.mapCenter);
  }
  get center() {
    return this.mapCenter;
  }

  setZoom(z: number) {
    this.mapZoom = z;
    this.zoomEmitter.next(this.mapZoom);
  }

  get zoom() {
    return this.mapZoom;
  }

  setEsriZoom(zoom: number) {
    this.esriZoom = zoom;
  }

  setEsriCenter(point: Point) {
    this.esriCenter = point;
  }
}

```

Super easy. An interface for an x/y coordinate, a number for a zoom level, and a center of the previous interface for an x/y coordinate.

Next we'll add some listeners to our `arcmap.component.ts`

```typescript
import { Component, OnInit, ViewChild, ElementRef } from '@angular/core';
import { loadModules } from 'esri-loader';
import esri = __esri;
import { ArcmapService, Point } from '../services/arcmap.service';

@Component({
  selector: 'app-arcmap',
  templateUrl: './arcmap.component.html',
  styleUrls: ['./arcmap.component.scss']
})
export class ArcmapComponent implements OnInit {
  @ViewChild('arcmap', { static: false })
  private arcMapRef: ElementRef;
  mapView: esri.MapView;
  map: esri.Map;

  constructor(private arcMapService: ArcmapService) {}

  ngOnInit() {
    this.init();
    this.arcMapService.zoomEmitter.subscribe((zoom: number) => {
      console.log('settnig zoom: ' + zoom);
      this.mapView.zoom = zoom;
    });
    this.arcMapService.centerEmitter.subscribe((center: Point) => {
      if (this.mapView.center.latitude !== center.lat || this.mapView.center.longitude !== center.lon) {
        this.mapView.goTo({
          center: [center.lon, center.lat]
        });
      }
    });
  }

  async initWidgets() {
    const [Search, BasemapGallery, Expand, Locate] = await loadModules([
      'esri/widgets/Search',
      'esri/widgets/BasemapGallery',
      'esri/widgets/Expand',
      'esri/widgets/Locate'
    ]);
    const locateWidget = new Locate({
      view: this.mapView
    });
    const basemapWidget = new BasemapGallery({ view: this.mapView });
    const searchWidget = new Search({
      view: this.mapView
    });
    this.mapView.ui.add(new Expand({ view: this.mapView, content: searchWidget }), 'top-right');
    this.mapView.ui.add(new Expand({ view: this.mapView, content: basemapWidget }), 'top-right');
    this.mapView.ui.add(locateWidget, 'top-right');
  }

  async init() {
    try {
      const [Map, MapView] = await loadModules(['esri/Map', 'esri/views/MapView']);
      this.map = new Map({
        basemap: 'hybrid'
      });
      this.mapView = new MapView({
        map: this.map,
        center: [-113.4909, 53.544],
        zoom: 12,
        container: this.arcMapRef.nativeElement,
        ui: {
          components: ['attribution']
        }
      });
      this.mapView.when(async () => {
        this.initWidgets();
      });
    } catch (e) {
      console.error(e);
    }
  }
}

```

In the `ngOnInit` we set up all our listeners to our map service including how to react to the events along with some quality of life updates for initializing some widgets and the like.

I guess it's about now that I should prove to you that this thing works hey...

I'm going to do this by making my app take you to a bunch of Canadian sports centers organized by city.

Under `app.component.ts` we'll need to add some data and a function to handle a click of this new data.  Here you go, a component is born:

```typescript
import { Component } from '@angular/core';
import { ArcmapService } from './services/arcmap.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'esri-app';
  isMenuOpen = true;

  navItems = [
    {
      title: 'Calgary',
      locations: [
        {
          title: 'Saddledome',
          location: {
            lat: 51.0374336,
            lon: -114.0519341
          },
          zoom: 18
        },
        {
          title: 'McMahon Stadium',
          location: {
            lat: 51.0703813,
            lon: -114.1214653
          },
          zoom: 18
        }
      ]
    },
    {
      title: 'Edmonton',
      locations: [
        {
          title: 'Rogers Place',
          location: {
            lat: 53.5469828,
            lon: -113.4979082
          },
          zoom: 19
        },
        {
          title: 'Commonwealth Stadium',
          location: {
            lat: 53.5596184,
            lon: -113.4761666
          },
          zoom: 19
        }
      ]
    },
    {
      title: 'Toronto',
      locations: [
        {
          title: 'Scotiabank Arena',
          location: {
            lat: 43.6434661,
            lon: -79.3790989
          },
          zoom: 20
        },
        {
          title: 'BMO Field',
          location: {
            lat: 43.6332247,
            lon: -79.4185654
          },
          zoom: 20
        }
      ]
    },
    {
      title: 'Vancouver',
      locations: [
        {
          title: 'Rogers Arena',
          location: {
            lat: 49.2778358,
            lon: -123.1088227
          },
          zoom: 21
        },
        {
          title: 'BC Place',
          location: {
            lat: 49.27675,
            lon: -123.111999
          },
          zoom: 21
        }
      ]
    }
  ];
  constructor(private service: ArcmapService) {}
  onMenuToggle() {
    this.isMenuOpen = !this.isMenuOpen;
  }
  onGotoLocation(location: { lat: number; lon: number }, zoom: number) {
    this.service.setZoom(zoom);
    this.service.setCenter(location);
  }
}


```

Next to update the template to be useful

```html
<div fxLayout="row" fxFlexFill>
  <mat-drawer-container class="example-container" fxLayout="column" fxFlex>
    <mat-drawer mode="side" [opened]="isMenuOpen" class="app-sidenav">
      <mat-card>
        <mat-card-title>Sports Venues</mat-card-title>
        <mat-card-subtitle>A small list</mat-card-subtitle>
      </mat-card>
      <mat-nav-list dense>
        <span *ngFor="let i of navItems">
          <h3 matSubheader>{{ i.title }}</h3>
          <a mat-list-item *ngFor="let n of i.locations" (click)="onGotoLocation(n.location, n.zoom)">
            {{ n.title }}
          </a>
        </span>
      </mat-nav-list>
    </mat-drawer>
    <mat-drawer-content fxLayout="column">
      <mat-toolbar fxLayout="row" color="primary">
        <button mat-icon-button (click)="onMenuToggle()">
          <mat-icon>menu</mat-icon>
        </button>
        <h3>ESRI Starter</h3>
      </mat-toolbar>
      <div fxFlex fxLayout="row">
        <div class="floating-panel">
          <router-outlet></router-outlet>
        </div>
        <app-arcmap fxFlex></app-arcmap>
      </div>
    </mat-drawer-content>
  </mat-drawer-container>
</div>


```

Go ahead and refresh the app and you can browse some of Canada's finest sports locales!

My short time working with the ArcGIS JavaScript API has been an interesting one.  It's impressively powerful and transitioned my opinions on GIS from 'So it's a map with some lines' to 'LOOK AT ALL THE COOL THINGS'.

The code on github has some more little knick-knacks added in. Feel free to explore and as questions! Especially around why I track two different map zoom levels and extents ;)

