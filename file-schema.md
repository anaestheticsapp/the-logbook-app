```ts
export interface LogbookFileFormatSchema {
  activity: {
    context: "theatre" | "icm" | "clinic" | "pain" | "procedure" | "session";
    destination?: "Day Case" | "Ward" | "POCU" | "Critical Care";
    priority?: "Elective" | "Urgent" | "Expedited" | "Immediate";
    diagnosis?: string;
    referral?: string;
    type?: string;
  };
  event: {
    anaesthesia?: {
      airway?: ("LMA" | "ETT" | "DLT")[];
      name: "GA" | "Sedation";
      options?: ("Volatile" | "TIVA" | "RSI")[];
    };
    incidents?: {
      name: string;
    }[];
    procedures?: {
      name: string;
      options?: ("Landmark" | "Ultrasound" | "Observed")[];
      category?: string;
      outcome?: string;
      notes?: string;
      supervision?: string;
    }[];
    regional?: {
      name: string;
      options?: ("Landmark" | "Ultrasound" | "Nerve Stimulator" | "Catheter" | "Observed")[];
      category?: string;
      outcome?: string;
      notes?: string;
      supervision?: string;
    }[];
    support?: {
      respiratory?: ("HFNO" | "NIV" | "IPPV" | "ECMO")[];
      cardiovascular?: ("Intra-aortic balloon pump" | "Pacing" | "Ventricular assist device")[];
      neuro?: ("EEG" | "ICP monitoring")[];
      renal?: [];
      liver?: [];
    }[];
    surgeries?: {
      name: string;
      speciality: string;
    }[];
  };
  metadata: {
    env?: "development" | "stable";
    id: string;
    name: string;
    timestamp: {
      start: number;
      inserted: number;
      edited?: number;
    };
    version: number;
  };
  note: string;
  patient: {
    age: number;
    ageUnits: "Days" | "Months" | "Years";
    ageCategory?: number;
    asa?: 1 | 2 | 3 | 4 | 5 | 6;
    sex?: "Male" | "Female";
  };
  setting: {
    country?: string;
    location?: string;
    rotation?: string;
  };
  timing: {
    date: string;
    session: "Morning" | "Afternoon" | "Evening" | "Night";
    duration?: number;
  };
  training: {
    supervision?: string;
    supervisor?: string;
    supervisorName?: string;
    teaching?: string;
  };
}
```
