# Drawing Grid

## Installation
`npm install ngx-drawing-grid`

## Usage
```typescript
import { NgModule } from '@angular/core';
import { DrawingGridModule } from 'ngx-drawing-grid';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [DrawingGridModule],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

```typescript
import { Component, ElementRef, OnInit } from '@angular/core';
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';
import { DrawingGridService, Pixel, PaintingMode } from 'drawing-grid';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss'],
})
export class AppComponent implements OnInit {
  private readonly destroy$: Subject<void> = new Subject<void>();

  width: number;
  height: number;
  pixelSize = 28;

  private paintingMode: PaintingMode;

  constructor(
    private host: ElementRef,
    private gridService: DrawingGridService,
  ) {}

  ngOnInit() {
    this.gridService.paintingMode$.pipe(takeUntil(this.destroy$)).subscribe((paintingMode) => {
      this.paintingMode = paintingMode;
    });

    this.width = this.host.nativeElement.clientWidth;
    this.height = this.host.nativeElement.clientHeight;
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }

  onMouseDown(pixel: Pixel) {
    this.fillPixel(pixel.x, pixel.y);
  }

  onMouseMove(pixel: Pixel) {
    this.fillPixel(pixel.x, pixel.y);
  }

  onMouseUp(pixel: Pixel) {}

  onContextMenu(pixel: Pixel) {
    this.gridService.clearPixel(pixel.x, pixel.y);
  }

  private fillPixel(x: number, y: number) {
    if (this.paintingMode === PaintingMode.ERASE) {
      this.gridService.clearPixel(x, y);
      return;
    }

    this.gridService.fillPixel(x, y, 'black');
  }
}
```
```html
<ng-container *ngIf="width && height">
  <drawing-grid
    [width]="width"
    [height]="height"
    [pixelSize]="pixelSize"
    (mouseDown)="onMouseDown($event)"
    (mouseMove)="onMouseMove($event)"
    (mouseUp)="onMouseUp($event)"
    (contextMenu)="onContextMenu($event)">
  </drawing-grid>
</ng-container>
```

## DrawingGridComponent Inputs
* `width` - The width of the canvas. The value will also be used for calculating the amount of pixels on the x-axis if the input `xNodes` is undefined
* `height` - The height of the canvas. The value will also be used for calculating the amount of pixels on the y-axis if the input `yNodes` is undefined
* `xNodes` - The amount of pixels on the y-axis
* `yNodes` - The amount of pixels on the y-axis
* `pixelSize` - The size of a pixel
* `fillStyle` - The fillstyle of the grid
* `disabled` - Disables or enables the canvas. If it is set to true the events will not get emitted

# DrawingGridComponent Output Events
* `mouseDown` - Gets emitted when the mouse has been pressed
* `mouseMove` - Gets emitted when the mouse has been pressed and is moving
* `mouseUp` - Gets emitted when the mouse has been released
* `contextMenu` - Gets emitted when the right mouse button has been pressed

Each output event from the `Drawing Grid Component` will return the current Pixel where the mouse is located on

## DrawingGridService API
* `isMouseLocked$` - Observe whether the mouse is currently locked or not. If the value is true either the left or right mouse button is currently pressed by the user
* `paintingMode$` - Observe the current set painting mode
* `pixels$` - Observe the pixels of the drawing grid
* `fillPixel()` - Update the fillstyle of a specific pixel
* `clearPixel()` - Resets the fillstyle of a specific pixel
* `getPixel()` - Returns the pixel at the given x and y coordinates
* `getPixelById()`- Returns the pixel which is associated with the given id
