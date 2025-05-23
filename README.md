# parcial-open

//Comandos angular
•	Crear proyecto
ng new nombre-proyecto –-skip-git

•	Agregar dependencias
ng add @angular/material (angular material componentes: botones, cards, etc)
npm install @ngx-translate/core @ngx-translate/http-loader --save (traduccion)
npm install -g json-server@0.17.4
•	Crear componentes en public (header, footer, language-switcher)
ng generate component public/components/header-content --skip-tests=true
ng generate component public/components/footer-content --skip-tests=true
ng generate component public/components/language-switcher --skip-tests=true

•	Crear environments
ng generate environments

•	Crear interface:
ng generate interface persons/services/person response

•	Crear Api //agregue esta vaina, me parece q falta esto p no?
ng generate service persons/services/persons-api --skip-tests=true

•	Crear model //los necesarios
ng generate class persons/model/person --type=entity --skip-tests=true


•	Crear assembler
ng generate class persons/services/person --type=assembler --skip-tests=true

•	Crear componentes
ng generate component persons/components/person-item --skip-tests=true
ng generate component persons/components/person-list --skip-tests=true


Ejecutar el siguiente command para iniciar el json-server:
json-server --watch server/db.json --routes server/routes.json

Ejecutar para iniciar el proyecto:
ng serve


//CODIGO	

routes.json

{
  "/api/v1/*": "/$1"
}


efficiency-item.component.css

.efficiency-card {
  margin: 20px 0;
}

.efficiency-card mat-card-header {
  background-color: #e8f4ff;
}

.efficiency-card mat-card-title {
  font-weight: bold;
}

.efficiency-card mat-card-subtitle {
  font-style: italic;
}

.efficiency-card mat-card-content {
  padding: 10px 15px;
}

efficiency-item.component.html
<mat-card class="efficiency-card">
  <mat-card-header>
    <mat-card-title>{{ record.fuelTankType }}</mat-card-title>
  </mat-card-header>

  <mat-card-content>
    <div><strong>{{ 'EFFICIENCY.BUSES_COUNT' | translate }}:</strong> {{ busesCount }}</div>
    <div><strong>{{ 'EFFICIENCY.AVERAGE_KM_PER_GALLON' | translate }}:</strong> {{ averageKmPerGallon }}</div>
  </mat-card-content>

  <mat-card-footer>
    <div><strong>{{ 'EFFICIENCY.REPORTED_ISSUES' | translate }}:</strong> {{ reportedIssues }}</div>
    <div><strong>{{ 'EFFICIENCY.LAST_REPORT' | translate }}:</strong> {{ lastReport }}</div>
  </mat-card-footer>
</mat-card>

efficiency-item.component.ts	
import { Component, Input, OnInit } from '@angular/core';
import { EfficiencyRecord } from '../../models/efficiency-record.entity';
import { Issue } from '../../models/issue.entity';
import { MatCardModule } from '@angular/material/card';
import { TranslatePipe } from '@ngx-translate/core';
import { NgIf, DatePipe } from '@angular/common';
import { FirstStudentAssembler } from '../../services/first-student.assembler';
@Component({
  selector: 'app-efficiency-item',
  standalone: true,
  imports: [MatCardModule, TranslatePipe, NgIf, DatePipe],
  templateUrl: './efficiency-item.component.html',
  styleUrls: ['./efficiency-item.component.css']
})
export class EfficiencyItemComponent implements OnInit {
  @Input() record!: EfficiencyRecord;
  @Input() issues: Issue[] = [];
  @Input() allRecords: EfficiencyRecord[] = [];

  busesCount = 0;
  averageKmPerGallon = 0;
  reportedIssues = 0;
  lastReport = 'No issues';



  ngOnInit(): void {
    const fuelType = this.record.fuelTankType;

    // Buses Count
    const buses = this.allRecords.filter(r => r.fuelTankType === fuelType);
    this.busesCount = buses.length;

    // Average Km Per Gallon
    const totalKm = buses.reduce((acc, r) => acc + r.averageKmPerGallon, 0);
    this.averageKmPerGallon = parseFloat((totalKm / (buses.length || 1)).toFixed(2));

    // Reported Issues
    const fuelIssues = this.issues.filter(i =>
      i.issueType === 'Fuel Tank Issue' &&
      this.allRecords.find(r => r.busId === i.busId)?.fuelTankType === fuelType
    );
    this.reportedIssues = fuelIssues.length;

    // Last Report
    if (fuelIssues.length > 0) {
      const latest = fuelIssues.reduce((a, b) =>
        new Date(a.registeredAt) > new Date(b.registeredAt) ? a : b
      );
      this.lastReport = new Date(latest.registeredAt).toLocaleString();
    }
  }


}
efficiency-list.component.css
.efficiency-list-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(400px, 1fr));
  gap: 11px;
}

@media (max-width: 600px) {
  .efficiency-list-container {
    grid-template-columns: 1fr;
  }
}

efficiency-list.component.html
<div class="efficiency-list-container">
  @for (record of records; track record.id) {
    <app-efficiency-item
      [record]="record"
      [issues]="issues"
      [allRecords]="records"
    />
  }
</div>

efficiency-list.component.ts
import { Component, OnInit } from '@angular/core';
import { EfficiencyRecord } from '../../models/efficiency-record.entity';
import { FirstStudentApiService } from '../../services/first-student-api.service';
import { EfficiencyItemComponent } from '../efficiency-item/efficiency-item.component';
import { NgFor } from '@angular/common';
import { Issue } from '../../models/issue.entity';

@Component({
  selector: 'app-efficiency-list',
  standalone: true,
  imports: [EfficiencyItemComponent, NgFor],
  templateUrl: './efficiency-list.component.html',
  styleUrls: ['./efficiency-list.component.css']
})
export class EfficiencyListComponent implements OnInit {
  records: EfficiencyRecord[] = [];
  issues: Issue[] = [];

  constructor(private apiService: FirstStudentApiService) {}

  ngOnInit(): void {
    this.apiService.getEfficiencyRecords().subscribe(
      (data) => this.records = data
    );
    this.apiService.getIssues().subscribe(
      (data) => this.issues = data
    );
  }
}
efficiency-record.entity.ts
export interface EfficiencyRecord {
  id: number;
  busId: string;
  fuelTankType: string;
  averageKmPerGallon: number;
  calculatedAt: string;
}

issue.entity.ts
export interface Issue {
  id: number;
  busId: string;
  issueType: string;
  registeredAt: string;
}

threshold.entity.ts
export interface Threshold {
  id: number;
  fuelTankType: string;
  minAverage: number;
  maxAverage: number;
}
first-student.assembler.ts
import { EfficiencyRecord } from '../models/efficiency-record.entity';
import { Threshold } from '../models/threshold.entity';
import { Issue } from '../models/issue.entity';

export class FirstStudentAssembler {
  static toEfficiencyRecord(dto: any): EfficiencyRecord {
    return {
      calculatedAt: "",
      id: dto.id,
      busId: dto.busId,
      fuelTankType: dto.fuelTankType,
      averageKmPerGallon: dto.averageKmPerGallon
    };
  }

  static toEfficiencyRecords(response: any[]): EfficiencyRecord[] {
    return response.map(dto => this.toEfficiencyRecord(dto));
  }

  static toIssue(dto: any): Issue {
    return {
      id: dto.id,
      busId: dto.busId,
      issueType: dto.issueType,
      registeredAt: dto.registeredAt
    };
  }

  static toIssues(response: any[]): Issue[] {
    return response.map(dto => this.toIssue(dto));
  }

  static toThreshold(dto: any): Threshold {
    return {
      maxAverage: 0, minAverage: 0,
      id: dto.id,
      fuelTankType: dto.fuelTankType,
    };
  }

  static toThresholds(response: any[]): Threshold[] {
    return response.map(dto => this.toThreshold(dto));
  }
}

first-student.response.ts
import { EfficiencyRecord } from '../models/efficiency-record.entity';
import { Threshold } from '../models/threshold.entity';
import { Issue } from '../models/issue.entity';

export interface FirstStudentResponse {
  records: EfficiencyRecord[];
  thresholds: Threshold[];
  issues: Issue[];
}

first-student-api.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, map } from 'rxjs';
import { EfficiencyRecord } from '../models/efficiency-record.entity';
import { Threshold } from '../models/threshold.entity';
import { Issue } from '../models/issue.entity';
import { environment } from "../../../environments/environment";
import { FirstStudentAssembler } from './first-student.assembler';

@Injectable({
  providedIn: 'root'
})
export class FirstStudentApiService {
  private baseUrl = environment.serverBasePath;

  constructor(private http: HttpClient) {}

  getEfficiencyRecords(): Observable<EfficiencyRecord[]> {
    return this.http.get<any[]>(`${this.baseUrl}${environment.efficiencyRecordsEndpointPath}`)
      .pipe(
        map(response => FirstStudentAssembler.toEfficiencyRecords(response))
      );
  }

  getThresholds(): Observable<Threshold[]> {
    return this.http.get<any[]>(`${this.baseUrl}${environment.thresholdsEndpointPath}`)
      .pipe(
        map(response => FirstStudentAssembler.toThresholds(response))
      );
  }

  getIssues(): Observable<Issue[]> {
    return this.http.get<any[]>(`${this.baseUrl}${environment.issuesEndpointPath}`)
      .pipe(
        map(response => FirstStudentAssembler.toIssues(response))
      );
  }

  addEfficiencyRecord(record: Partial<EfficiencyRecord>): Observable<EfficiencyRecord> {
    return this.http.post<EfficiencyRecord>(`${this.baseUrl}${environment.efficiencyRecordsEndpointPath}`, record);
  }

  addIssue(issue: Partial<Issue>): Observable<Issue> {
    return this.http.post<Issue>(`${this.baseUrl}${environment.issuesEndpointPath}`, issue);
  }
}
//HEADER Y MAS

header-content.component.css
mat-toolbar {
  background-color: #e0e0e0;
  padding: 0 16px;
}


.toolbar {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  gap: 10px;
}


.toolbar-left {
  display: flex;
  align-items: center;
  flex-shrink: 0;
  gap: 10px;
}

.logo {
  height: 40px;
  width: auto;
  max-width: 100%;
}

.toolbar-title {
  font-size: 24px;
  font-weight: 700;
  color: #212121;
  white-space: nowrap;
}


.toolbar-center {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
  flex: 1 1 auto;
}


.toolbar-right {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: flex-end;
  flex-shrink: 0;
}


@media (max-width: 768px) {
  .toolbar {
    flex-direction: column;
    align-items: stretch;
    text-align: center;
  }

  .toolbar-left,
  .toolbar-center,
  .toolbar-right {
    width: 100%;
    justify-content: center;
  }

  .toolbar-title {
    font-size: 20px;
  }

  .logo {
    height: 32px;
  }
}

header-content.component.html
<mat-toolbar color="primary">
  <div class="toolbar">
    <div class="toolbar-left">
      <img src="https://logo.clearbit.com/firststudentinc.com" alt="FIRSTstudent Logo" class="logo" />
      <span class="toolbar-title">HALO Maintenance</span>
    </div>

    <div class="toolbar-center">
      <a mat-button [routerLink]="'/home'">{{ 'toolbar.home' | translate }}</a>
      <a mat-button [routerLink]="'/maintenance/efficiency-records/new'">
        {{ 'toolbar.fuel' | translate }}
      </a>
    </div>

    <div class="toolbar-right">
      <app-language-switcher></app-language-switcher>
    </div>
  </div>
</mat-toolbar>

header-content.component.ts
import { Component } from '@angular/core';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatButtonModule } from '@angular/material/button';
import { LanguageSwitcherComponent } from '../language-switcher/language-switcher.component';
import { RouterModule } from '@angular/router';
import { TranslateModule } from '@ngx-translate/core';

@Component({
  selector: 'app-header-content',
  standalone: true,
  imports: [
    MatToolbarModule,
    MatButtonModule,
    LanguageSwitcherComponent,
    RouterModule,
    TranslateModule,
  ],
  templateUrl: './header-content.component.html',
  styleUrls: ['./header-content.component.css']
})
export class HeaderContentComponent {}
language-switcher.component.css
language-switcher.component.html
<mat-button-toggle-group [value]="currentLang" appearance="standard" aria-label="Preferred Language" name="language">
  @for (language of languages; track language) {
    <mat-button-toggle (click)="useLanguage(language)"
                       [aria-label]="language"
                       [value]="language">{{ language.toUpperCase() }}
    </mat-button-toggle>
  }
</mat-button-toggle-group>

language-switcher.component.ts
import { Component } from '@angular/core';

import { TranslateService } from "@ngx-translate/core";
import { MatButtonToggleModule } from '@angular/material/button-toggle';

@Component({
  selector: 'app-language-switcher',
  imports: [MatButtonToggleModule],
  templateUrl: './language-switcher.component.html',
  styleUrl: './language-switcher.component.css'
})
export class LanguageSwitcherComponent {
  currentLang: string = 'en';
  languages: string[] = ['en', 'es'];

  constructor(private translate: TranslateService) {
    this.currentLang = translate.currentLang;
  }

  useLanguage(language: string) : void {
    this.translate.use(language);
  }
}
//PAGES Y MAS
fuel-efficiency.component.css
.fuel-efficiency-container {
  padding: 24px;
}

.add-record-section {
  margin-top: 24px;
  border: 1px solid #ccc;
  padding: 16px;
  border-radius: 8px;
}

mat-form-field {
  display: block;
  margin-bottom: 16px;
  width: 100%;
  max-width: 400px;
}

.message {
  margin-top: 16px;
  font-weight: bold;
  color: black;
}

fuel-efficiency.component.html
<div class="fuel-efficiency-container">
  <h1>{{ 'EFFICIENCY.TITLE' | translate }}</h1>

  <section class="add-record-section">
    <h2>{{ 'EFFICIENCY.SUBTITLE' | translate }}</h2>
    <form (ngSubmit)="addRecord()" #efficiencyForm="ngForm">
      <mat-form-field appearance="fill">
        <mat-label>{{ 'EFFICIENCY.BUS_ID' | translate }}</mat-label>
        <input matInput required name="busId" [(ngModel)]="busId">
      </mat-form-field>

      <mat-form-field appearance="fill">
        <mat-label>{{ 'EFFICIENCY.FUEL_TANK_TYPE' | translate }}</mat-label>
        <mat-select required name="fuelTankType" [(ngModel)]="fuelTankType">
          <mat-option value="Type A">{{ 'EFFICIENCY.TYPE_A' | translate }}</mat-option>
          <mat-option value="Type B">{{ 'EFFICIENCY.TYPE_B' | translate }}</mat-option>
          <mat-option value="Type C">{{ 'EFFICIENCY.TYPE_C' | translate }}</mat-option>
          <mat-option value="Type D">{{ 'EFFICIENCY.TYPE_D' | translate }}</mat-option>
        </mat-select>
      </mat-form-field>

      <mat-form-field appearance="fill">
        <mat-label>{{ 'EFFICIENCY.AVG_KM' | translate }}</mat-label>
        <input matInput required type="number" name="averageKmPerGallon" [(ngModel)]="averageKmPerGallon">
      </mat-form-field>

      <button mat-raised-button color="primary" type="submit">
        {{ 'EFFICIENCY.ADD_BUTTON' | translate }}
      </button>
    </form>

    <div class="message">
      {{ message }}
    </div>
  </section>
</div>

fuel-efficiency.component.ts

import { Component } from '@angular/core';
import { FirstStudentApiService } from '../../../firststudent/services/first-student-api.service';
import { EfficiencyRecord } from '../../../firststudent/models/efficiency-record.entity';
import { Threshold } from '../../../firststudent/models/threshold.entity';
import { Issue } from '../../../firststudent/models/issue.entity';
import { firstValueFrom } from 'rxjs';

import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
import { MatOptionModule } from '@angular/material/core';
import { MatButtonModule } from '@angular/material/button';
import { FormsModule } from '@angular/forms';

import { TranslateModule } from '@ngx-translate/core';
@Component({
  selector: 'app-fuel-efficiency',
  templateUrl: './fuel-efficiency.component.html',
  imports: [
    FormsModule,
    MatFormFieldModule,
    MatInputModule,
    MatSelectModule,
    MatOptionModule,
    MatButtonModule,
    TranslateModule
  ],
  styleUrls: ['./fuel-efficiency.component.css']
})
export class FuelEfficiencyComponent {
  busId: string = '';
  fuelTankType: string = '';
  averageKmPerGallon: number | null = null;
  message: string = '';
  isError: boolean = false;

  constructor(private apiService: FirstStudentApiService) {}

  async addRecord(): Promise<void> {
    // Validación simple antes de enviar
    if (!this.busId.trim() || !this.fuelTankType.trim() || this.averageKmPerGallon === null || this.averageKmPerGallon <= 0) {
      this.message = 'Please fill all fields correctly.';
      this.isError = true;
      return;
    }

    try {
      const thresholds = await firstValueFrom(this.apiService.getThresholds());
      const threshold = thresholds.find(t => t.fuelTankType === this.fuelTankType);

      if (!threshold) {
        this.message = 'Invalid Fuel Tank Type';
        this.isError = true;
        return;
      }

      const newRecord: EfficiencyRecord = {
        id: Date.now(),
        busId: this.busId,
        fuelTankType: this.fuelTankType,
        averageKmPerGallon: this.averageKmPerGallon,
        calculatedAt: new Date().toISOString()
      };

      await firstValueFrom(this.apiService.addEfficiencyRecord(newRecord));

      const isOutOfBounds =
        newRecord.averageKmPerGallon < threshold.minAverage ||
        newRecord.averageKmPerGallon > threshold.maxAverage;

      if (isOutOfBounds) {
        const issues = await firstValueFrom(this.apiService.getIssues());
        const today = new Date().toISOString().split('T')[0];

        const alreadyReportedToday = issues.some(issue =>
          issue.busId === this.busId &&
          issue.issueType === 'Fuel Tank Issue' &&
          issue.registeredAt.startsWith(today)
        );

        if (!alreadyReportedToday) {
          const newIssue: Issue = {
            id: Date.now(),
            busId: this.busId,
            issueType: 'Fuel Tank Issue',
            registeredAt: new Date().toISOString()
          };

          await firstValueFrom(this.apiService.addIssue(newIssue));
        }
      }

      this.message = 'Record successfully created';
      this.isError = false;
      this.resetForm();
    } catch (error) {
      this.message = 'An error occurred while adding the record.';
      this.isError = true;
    }
  }

  resetForm(): void {
    this.busId = '';
    this.fuelTankType = '';
    this.averageKmPerGallon = null;
  }
}
home.component.css
h1 {
  font-size: 28px;
  margin-bottom: 10px;
}

.efficiency-section {
  margin-top: 30px;
}

home.component.html
<h1>{{ 'home.home' | translate }}</h1>
<p>{{ 'home.welcome' | translate }}</p>

<section class="efficiency-section">
  <h2>{{ 'home.efficiency' | translate }}</h2>
  <app-efficiency-list></app-efficiency-list>
</section>

home.component.ts
import { Component } from '@angular/core';
import { TranslateModule } from '@ngx-translate/core';
import {MatToolbarModule} from '@angular/material/toolbar';
import {MatButtonModule} from '@angular/material/button';
import {LanguageSwitcherComponent} from '../../components/language-switcher/language-switcher.component';
import {RouterModule} from '@angular/router';
import {EfficiencyListComponent} from '../../../firststudent/components/efficiency-list/efficiency-list.component';

@Component({
  selector: 'app-home',
  imports: [
    MatToolbarModule,
    MatButtonModule,
    LanguageSwitcherComponent,
    RouterModule,
    TranslateModule,
    EfficiencyListComponent,
  ],
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent {}
page-not-found.component.css
.not-found-container {
  text-align: center;
  margin-top: 100px;
  padding: 20px;
}

.not-found-container h1 {
  font-size: 2.5em;
  color: #c62828;
}

.not-found-container p {
  font-size: 1.2em;
  margin: 20px 0;
}

button {
  font-size: 1em;
}

page-not-found.component.html
<div class="not-found-container">
  <h1>404 - Page Not Found</h1>
  <p>The path <strong>{{ path }}</strong> does not exist.</p>
  <button mat-raised-button color="primary" (click)="goHome()">Go to Home</button>
</div>

page-not-found.component.ts
import { Component } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';

@Component({
  selector: 'app-page-not-found',
  templateUrl: './page-not-found.component.html',
  styleUrls: ['./page-not-found.component.css']
})
export class PageNotFoundComponent {
  path: string | null = '';

  constructor(private route: ActivatedRoute, private router: Router) {
    const url = this.router.url;
    this.path = url;
  }

  goHome(): void {
    this.router.navigate(['/home']);
  }
}
app.component.html
<div class="app-container">
  <app-header-content/>
  <router-outlet></router-outlet>
</div>
app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

import { TranslateService } from '@ngx-translate/core';
import {HeaderContentComponent} from './public/components/header-content/header-content.component';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet,HeaderContentComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {
  title = 'HALO Maintenance';
  constructor(private translate: TranslateService) {
    this.translate.addLangs(['en', 'es']);
    this.translate.setDefaultLang('en');
    this.translate.use('en');
  }
}
app.config.ts
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';

import { HttpClient, provideHttpClient } from "@angular/common/http";
import { routes } from './app.routes';
import { provideTranslateService, TranslateLoader } from "@ngx-translate/core";
import { TranslateHttpLoader } from "@ngx-translate/http-loader";


const httpLoaderFactory: (http: HttpClient) =>
  TranslateLoader = (http: HttpClient) =>
  new TranslateHttpLoader(http, './assets/i18n/', '.json');

export const appConfig: ApplicationConfig = {
  providers: [provideZoneChangeDetection({ eventCoalescing: true }), provideRouter(routes),
    provideHttpClient(),
    provideTranslateService({
      loader: {
        provide: TranslateLoader,
        useFactory: httpLoaderFactory,
        deps: [HttpClient]
      },
      defaultLanguage: 'en',
    })
  ]
};
app.routes.ts
import { Routes } from '@angular/router';
import { HomeComponent } from './public/pages/home/home.component';
import { FuelEfficiencyComponent } from './public/pages/fuel-efficiency/fuel-efficiency.component';
import { PageNotFoundComponent } from './public/pages/page-not-found/page-not-found.component';

export const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'maintenance/efficiency-records/new', component: FuelEfficiencyComponent },
  { path: '**', component: PageNotFoundComponent },
];
//Enviroments
environment.development.ts
export const environment = {
  production: false,
  serverBasePath: 'http://localhost:3000/api/v1',
  thresholdsEndpointPath: '/thresholds',
  efficiencyRecordsEndpointPath: '/efficiency-records',
  issuesEndpointPath: '/issues'
};

environment.ts

export const environment = {
  production: true,
  serverBasePath: 'http://localhost:3000/api/v1',
  thresholdsEndpointPath: '/thresholds',
  efficiencyRecordsEndpointPath: '/efficiency-records',
  issuesEndpointPath: '/issues'
};
