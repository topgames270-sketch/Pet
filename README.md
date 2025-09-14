-- users e roles
CREATE TABLE roles (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL -- e.g., admin, groomer, receptionist
);

CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  role_id INT REFERENCES roles(id),
  name VARCHAR(150) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  phone VARCHAR(30),
  created_at TIMESTAMP DEFAULT now()
);

-- clients e animals
CREATE TABLE clients (
  id SERIAL PRIMARY KEY,
  name VARCHAR(150) NOT NULL,
  email VARCHAR(255),
  phone VARCHAR(30),
  address TEXT,
  created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE animals (
  id SERIAL PRIMARY KEY,
  client_id INT REFERENCES clients(id) ON DELETE CASCADE,
  name VARCHAR(100),
  species VARCHAR(50),
  breed VARCHAR(100),
  sex VARCHAR(10),
  birth_date DATE,
  weight_kg NUMERIC(5,2),
  notes TEXT,
  photo_url TEXT,
  created_at TIMESTAMP DEFAULT now()
);

-- services e checklists
CREATE TABLE services (
  id SERIAL PRIMARY KEY,
  name VARCHAR(150) NOT NULL, -- ex: Banho + Tosa
  description TEXT,
  base_price NUMERIC(10,2) DEFAULT 0
);

CREATE TABLE checklist_templates (
  id SERIAL PRIMARY KEY,
  service_id INT REFERENCES services(id) ON DELETE CASCADE,
  name VARCHAR(150) NOT NULL, -- ex: Banho Básico - Checklist
  description TEXT
);

CREATE TABLE checklist_items (
  id SERIAL PRIMARY KEY,
  template_id INT REFERENCES checklist_templates(id) ON DELETE CASCADE,
  position INT DEFAULT 0,
  label VARCHAR(255) NOT NULL,
  type VARCHAR(20) DEFAULT 'boolean', -- boolean, text, number, photo, select
  options TEXT, -- JSON or comma separated for select
  required BOOLEAN DEFAULT false
);

-- appointments (agendamentos)
CREATE TABLE appointments (
  id SERIAL PRIMARY KEY,
  client_id INT REFERENCES clients(id),
  animal_id INT REFERENCES animals(id),
  service_id INT REFERENCES services(id),
  scheduled_at TIMESTAMP,
  status VARCHAR(30) DEFAULT 'scheduled', -- scheduled, in_service, completed, canceled
  assigned_user_id INT REFERENCES users(id),
  total_price NUMERIC(10,2),
  created_at TIMESTAMP DEFAULT now()
);

-- checklist responses
CREATE TABLE checklist_responses (
  id SERIAL PRIMARY KEY,
  appointment_id INT REFERENCES appointments(id) ON DELETE CASCADE,
  template_id INT REFERENCES checklist_templates(id),
  created_by INT REFERENCES users(id),
  created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE checklist_answers (
  id SERIAL PRIMARY KEY,
  response_id INT REFERENCES checklist_responses(id) ON DELETE CASCADE,
  item_id INT REFERENCES checklist_items(id),
  value TEXT, -- store true/false, text or JSON (for photos: URL)
  note TEXT
);

-- payments, attachments, audit
CREATE TABLE payments (
  id SERIAL PRIMARY KEY,
  appointment_id INT REFERENCES appointments(id),
  amount NUMERIC(10,2),
  method VARCHAR(50),
  paid_at TIMESTAMP DEFAULT now()
);

CREATE TABLE attachments (
  id SERIAL PRIMARY KEY,
  appointment_id INT REFERENCES appointments(id),
  url TEXT,
  type VARCHAR(50),
  uploaded_by INT REFERENCES users(id),
  uploaded_at TIMESTAMP DEFAULT now()
);

CREATE TABLE audit_logs (
  id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(id),
  action VARCHAR(255),
  entity VARCHAR(100),
  entity_id INT,
  data JSONB,
  created_at TIMESTAMP DEFAULT now()
);
