-- feedback-site Supabase tables (spec: .scratch/feedback-site/spec.md)
-- Supabase SQL Editor > New query > paste all > Run. Safe to run again.

-- 1. students
create table if not exists students (
  id uuid primary key default gen_random_uuid(),
  class_no int not null,
  student_no int not null,
  name text not null,
  pin text check (pin ~ '^[0-9]{4}$'),
  created_at timestamptz not null default now(),
  unique (class_no, student_no)
);

-- 2. questions (latest row = current question)
create table if not exists questions (
  id uuid primary key default gen_random_uuid(),
  body text not null check (length(trim(body)) > 0),
  created_at timestamptz not null default now()
);

-- 3. submissions (one per student per question)
create table if not exists submissions (
  id uuid primary key default gen_random_uuid(),
  question_id uuid not null references questions(id),
  student_id uuid not null references students(id),
  answer text not null check (length(trim(answer)) > 0),
  difficulty text,
  feedback text,
  is_read boolean not null default false,
  reply text,
  created_at timestamptz not null default now(),
  unique (question_id, student_id)
);

-- 4. starting data
insert into students (class_no, student_no, name) values (1, 1, '하늘') on conflict (class_no, student_no) do nothing;
insert into students (class_no, student_no, name) values (1, 2, '바다') on conflict (class_no, student_no) do nothing;
insert into students (class_no, student_no, name) values (1, 3, '숲') on conflict (class_no, student_no) do nothing;
insert into students (class_no, student_no, name) values (1, 4, '별') on conflict (class_no, student_no) do nothing;
insert into students (class_no, student_no, name) values (1, 5, '구름') on conflict (class_no, student_no) do nothing;

insert into questions (body) select '오늘 수업에서 배운 전류와 전압은 어떻게 다른가요? 친구에게 설명하듯 내 말로 2~3문장 써 보세요.' where not exists (select 1 from questions);

-- 5. RLS: anon can read, insert, update. No delete policy, so delete is blocked.
alter table students enable row level security;
alter table questions enable row level security;
alter table submissions enable row level security;

drop policy if exists "practice read students" on students;
create policy "practice read students" on students for select to anon, authenticated using (true);
drop policy if exists "practice insert students" on students;
create policy "practice insert students" on students for insert to anon, authenticated with check (true);
drop policy if exists "practice update students" on students;
create policy "practice update students" on students for update to anon, authenticated using (true) with check (true);

drop policy if exists "practice read questions" on questions;
create policy "practice read questions" on questions for select to anon, authenticated using (true);
drop policy if exists "practice insert questions" on questions;
create policy "practice insert questions" on questions for insert to anon, authenticated with check (true);
drop policy if exists "practice update questions" on questions;
create policy "practice update questions" on questions for update to anon, authenticated using (true) with check (true);

drop policy if exists "practice read submissions" on submissions;
create policy "practice read submissions" on submissions for select to anon, authenticated using (true);
drop policy if exists "practice insert submissions" on submissions;
create policy "practice insert submissions" on submissions for insert to anon, authenticated with check (true);
drop policy if exists "practice update submissions" on submissions;
create policy "practice update submissions" on submissions for update to anon, authenticated using (true) with check (true);
