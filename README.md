import {AfterViewInit, Component, OnDestroy, OnInit} from '@angular/core';
import {animate, state, style, transition, trigger} from '@angular/animations';
import {StatisticService} from "../../shared/services/statistic.service";
import {DomSanitizer} from "@angular/platform-browser";
import {OtherService} from "../../shared/services/other.service";
import {DynamicTextService} from "../../shared/services/dynamic-text.service";

@Component({
  selector: 'app-main',
  templateUrl: './main.component.html',
  styleUrls: ['./main.component.scss'],
  animations: [
    trigger('slider', [
      state('resetSlide', style({
        transform: 'translateX(0)'
      })),
      state('nextSlide', style({
        transform: '{{changed}}'
      }), {params: {changed: 'translateX(100)'}}),
      state('nextSlide1', style({
        transform: '{{changed}}'
      }), {params: {changed: 'translateX(200)'}}),
      transition('resetSlide <=> nextSlide', animate('1000ms ease-in-out')),
      transition('nextSlide <=> nextSlide1', animate('1000ms ease-in-out')),
      transition('nextSlide1 <=> resetSlide', animate('1000ms ease-in-out')),
    ])
  ]

})
export class MainComponent implements OnInit {

  authorizen: boolean = false;
  state: string = 'resetSlide';

  register: boolean = false;

  statistic: any ={};

  transforming: string = '';

  count: number = 0;
  slides: number = 0;

  staticText: any = [];
  youtube: any;
  gallery: any = [];

  currentIndex = 0;
  speed = 5000;
  infinite = true;
  direction = 'right';
  directionToggle = true;
  autoplay = true;
  avatars = '1234567891234'.split('').map((x, i) => {
    const num = i;
    // const num = Math.floor(Math.random() * 1000);
    return {
      url: `https://picsum.photos/600/400/?${num}`,
      title: `${num}`
    };
  });

  constructor(public statisticService: StatisticService,
              public dynamicTextService: DynamicTextService,
              public otherService: OtherService,
              public sanitizer: DomSanitizer) {
  }

  ngOnInit() {
    if(localStorage.getItem('token') !== undefined){
      this.authorizen = true;
    }
    this.getSliderPhotos();
    this.getStaticText();
    this.slide();
  }

  getRegister(event){
    this.register = event;
  }

  getStaticText(){
    this.dynamicTextService.getAll()
      .subscribe(res =>{
        this.staticText = res['data'];
        this.statistic = this.staticText[9];
        console.log(this.staticText[9]);
        this.youtube = this.sanitizer.bypassSecurityTrustResourceUrl(this.staticText[1].field2);
      })
  }

  getSliderPhotos(){
    this.otherService.getSliderPhotos()
      .subscribe(res =>{
        this.gallery = res['success'];
        this.slide();
      })
  }

  auth(event){
    this.authorizen = event;
    console.log(this.authorizen);
  }

  toVideo(target){
    target.scrollIntoView({ behavior: 'smooth',block: 'center'});
  }

  slide(){
    setInterval(() => {
        if (this.slides === this.gallery.length - 1) {
          this.state = this.state === 'nextSlide' ? 'nextSlide1' : 'nextSlide';
          switch (this.state) {
            case 'nextSlide': {
              this.count -= 100;
              break;
            }
            case 'nextSlide1': {
              this.count -= 100;
              break;
            }
          }
          this.transforming = `translateX(${this.count}%)`;
          if(this.count === 0){
            this.slides = 0;
          }
        } else {
          this.state = this.state === 'nextSlide' ? 'nextSlide1' : 'nextSlide';
          this.slides += 1;
          switch (this.state) {
            case 'nextSlide': {
              this.count += 100;
              break;
            }
            case 'nextSlide1': {
              this.count += 100;
              break;
            }
          }
          this.transforming = `translateX(${this.count}%)`;
        }
    }, this.gallery[0].time);
  }

  paginateTo(index){
      this.count = (index * 100) - 100;
      this.slides = index - 1;
      this.state = this.state === 'nextSlide' ? 'nextSlide1' : 'nextSlide';
      this.transforming = `translateX(${this.count}%)`;

  }

  toRegister(header) {
    header.scrollIntoView({behavior: 'smooth', block: 'start'});
    this.register = true;
  }

}
