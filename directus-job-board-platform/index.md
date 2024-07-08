---
title: 'Building a Job Board Platform with Directus and SolidStart.js'
description: 'Learn how to build a job board portal with Directus and SolidStart.js. You'll learn how to register new users, login, and perform create, read, update, and delete operations (CRUD) on Directus.'
author:
  name: 'Ndoma Precious'
  avatar_file_name: './ndoma-precious.png'
---

In this tutorial, you'll learn to build a job board portal using Directus and SolidStart.js. We'll cover user registration, login, and working with data in Directus. You'll create a complete job board with listing, application, and management features for both jobs and applications. This guide will provide you with the skills to combine Directus's backend capabilities with SolidStart.js's reactive frontend, to build a job portal web application.

## Before You Start
You will need:

- [Node.js v18](https://nodejs.org/) and above installed on your computer.
- A Directus project - follow our [quickstart guide](https://docs.directus.io/getting-started/quickstart) if you don't already have one.
- Some experience with Typescript and SolidStart.js.

The code for this tutorial is available on this [GitHub repository](https://github.com/preshenv/directus-solidstart-job-board).

## Configuring Directus Data Models

Create a `job`, and `application` collection in your project. The `job` will store a list available jobs with the following fields:

- `id`: autocomplete input
- `title`: input field
- `location`: input field
- `type`: input field
- `salary`: input field

The `application` collection will store job applications with the following fields:

- `id:` Autocomplete Input
- `status`: input field

To link a new job with its creator:

1. In the `job` collection, add a Many-to-One field named `employer`.
2. Set the related collection to System > `directus_users`.
3. Choose `first_name` as the Display template.

This creates a relationship between the job and the admin who created it.

In the `application` collection, add two Many-to-One fields:

1. `user`: Links to the applicant
2. `job`: Links to the job

These fields connect each application to its corresponding applicant and job. Add 3 items in the `job` collection - [here's some sample data](https://raw.githubusercontent.com/preshenv/directus-solidstart-job-board/main/src/components/jobs.md).

## Creating a New User Role
In your access control settings, create a new role called `Job Applicant`. For the `application` collection, enable `create` and `read` permissions. Use custom rules for the `application` collection to ensure users can only read and update their own applications. Set a filter like: `user -> id Equals $CURRENT_USER.id`. This allows users to view all jobs, create new applications, and view or update only their own applications. 

## Allowing Public Registration
Open the public role, find `directus_users` under system collections, and then open custom permissions for the `create` operation. 
1. In field permissions, enable only `first_name`, `last_name`, `email`, and `password` options. If you want users to provide other data at the time of registration, enable the respective field.
2. In field validation, require `first_name`, `last_name`, `email`, and `password` to not be empty.
3. In field presets, set the new user's `role` to the `Job Applicant` role id. 

Give public view access to the `job` collection to allow users to see available jobs even when they are not logged in. Also, ensure the admin role retains full permissions across all collections.

## Initializing a SolidStart.js project

Create a new SolidStart project by running the command:

```env
npm init solid@latest
```

Choose the **bare** template, enable Server-side rendering from the prompt, and Typescript in the prompts.

In your SolidStart project's `src` directory, create a `lib` directory. Inside it, create a `directus.js` file:

```jsx
import { authentication, createDirectus, rest } from "@directus/sdk";

export const PUBLIC_DIRECTUS_API_URL = import.meta.env
  .VITE_PUBLIC_DIRECTUS_API_URL;

function getDirectusInstance() {
  const directus = createDirectus(PUBLIC_DIRECTUS_API_URL)
    .with(
      authentication("cookie", { credentials: "include", autoRefresh: true })
    )
    .with(rest({ credentials: "include" }));
  return directus;
}
export default getDirectusInstance;
```

Add your Directus URL to the `.env` file:

```env
VITE_PUBLIC_DIRECTUS_API_URL='https://directus.example.com'
```

## Implementing User Authentication
To implement user authentication to grant users access to the application, create a **context**, and inside that folder create an `AuthContext.tsx` file and add the following code:

```jsx
import {
  createContext,
  useContext,
  JSX,
  createSignal,
  createEffect,
} from "solid-js";
import { User } from "../types";
import getDirectusInstance from "~/lib/directus";
import { createUser, readMe, withToken } from "@directus/sdk";

const setCookie = (name: string, value: string, days: number = 7) => {
  const expires = new Date(Date.now() + days * 864e5).toUTCString();
  document.cookie = `${name}=${encodeURIComponent(
    value
  )}; expires=${expires}; path=/`;
};

const getCookie = (name: string): string | null => {
  return document.cookie.split("; ").reduce((r, v) => {
    const parts = v.split("=");
    return parts[0] === name ? decodeURIComponent(parts[1]) : r;
  }, "");
};

const deleteCookie = (name: string) => {
  setCookie(name, "", -1);
};

interface AuthContextType {
  user: () => User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  register: (user: Omit<User, "id">) => Promise<void>;
}

const AuthContext = createContext<AuthContextType>();
const directus = getDirectusInstance();

export function AuthProvider(props: { children: JSX.Element }) {
  const [user, setUser] = createSignal<User | null>(null);
  const getToken = () => getCookie("auth_token") || "";

  createEffect(() => {
    const token = getToken();
    if (token) {
      directus.setToken(token);
      fetchUser();
    }
  });

  const fetchUser = async () => {
    try {
      const userData = await directus.request(
        withToken(
          getToken(),
          readMe({
            fields: ["*"],
            deep: {
              role: {
                fields: ["*"],
              },
            },
          })
        )
      );
      setUser(userData as User);
    } catch (error) {
      await logout();
    }
  };

  const login = async (email: string, password: string) => {
    try {
      const result = await directus.login(email, password);
      setCookie("auth_token", result.access_token as string);
      directus.setToken(result.access_token);
      await fetchUser();
    } catch (error) {
      throw new Error("Invalid credentials");
    }
  };

  const logout = async () => {
    try {
      await directus.logout();
    } catch (error) {
      console.error("Logout error:", error);
    } finally {
      setUser(null);
      deleteCookie("auth_token");
      directus.setToken(null);
    }
  };

  const register = async (newUser: Omit<User, "id">) => {
    try {
      const result = await directus.request(
        createUser({
          email: newUser.email,
          password: newUser.password,
          first_name: newUser.name.split(" ")[0],
          last_name: newUser.name.split(" ")[1],
          role: "Job Applicant",
        })
      );
    } catch (error) {
      throw new Error("Registration failed");
    }
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, register }}>
      {props.children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => useContext(AuthContext)!;
```
This `AuthContext` handles the user's registration, login, logout, and session management which retrieves user session information from cookies, including access and refresh tokens, and returns an object containing this information.

In your **src** folder, create a new **types** directory. Add an index.ts file inside it to define the `User` interface used in `AuthContext` and other interfaces you'll be using throughout your application. This centralizes your TypeScript type definitions.

```jsx
export interface User {
  id: number;
  email: string;
  password: string;
  first_name: string;
  last_name: string;
  role?: string;
}

export interface Application {
  id: number;
  job: Job;
  user: User,
  status: 'pending' | 'reviewed' | 'accepted' | 'rejected';
  resumeUrl: string;
}

export interface Job {
  id?: number;
  title: string;
  description: string;
  location: string;
  type: string;
  salary: number;
  employer?: User | string;
}

export type Jobs = Job[];
export type Applications = Application[];
```

Create two new files, `register.tsx` and `login.tsx`, in your routes directory to implement the user registration and login forms. Add the following code to the `register.tsx` file.

```jsx
import { createSignal } from "solid-js";
import { useAuth } from "../context/AuthContext";
import { useNavigate } from "@solidjs/router";

export default function RegisterPage() {
  const [first_name, setFirstName] = createSignal("");
  const [last_name, setLastName] = createSignal("");
  const [email, setEmail] = createSignal("");
  const [password, setPassword] = createSignal("");
  const [role, setRole] = createSignal<"applicant" | "employer">("applicant");
  const auth = useAuth();
  const navigate = useNavigate();

  const handleRegister = async (e: Event) => {
    e.preventDefault();
    try {
     const res = await auth.register({
        first_name: first_name(),
        last_name: last_name(),
        email: email(),
        password: password(),
      });
      await auth.login(email(), password());
      navigate("/", { replace: true });
    } catch (err) {
      console.log(err)
      alert("Registration failed. Please try again.");
    }
  };

  return (
    <form onSubmit={handleRegister}>
      <input
        type="text"
        placeholder="First Name"
        value={first_name()}
        onInput={(e) => setFirstName(e.currentTarget.value)}
        required
      />
       <input
        type="text"
        placeholder="Last Name"
        value={last_name()}
        onInput={(e) => setLastName(e.currentTarget.value)}
        required
      />
      <input
        type="email"
        placeholder="Email"
        value={email()}
        onInput={(e) => setEmail(e.currentTarget.value)}
        required
      />
      <input
        type="password"
        placeholder="Password"
        value={password()}
        onInput={(e) => setPassword(e.currentTarget.value)}
        required
      />
      <select
        value={role()}
        onChange={(e) =>
          setRole(e.currentTarget.value as "applicant" | "employer")
        }
      >
        <option value="applicant">Applicant</option>
        <option value="employer">Employer</option>
      </select>
      <button type="submit">Register</button>
    </form>
  );
}
```
Then add the code snippets below to the `login.tsx` file.

```jsx
import { createSignal } from "solid-js";
import { useAuth } from "../context/AuthContext";
import { useNavigate } from "@solidjs/router";

export default function LoginPage() {
  const [email, setEmail] = createSignal("");
  const [password, setPassword] = createSignal("");
  const navigate = useNavigate();
  const auth = useAuth();

  const handleLogin = async (e: Event) => {
    e.preventDefault();
    try {
      await auth.login(email(), password());
      navigate("/", { replace: true });
    } catch (err) {
      alert("Invalid credentials. Please try again.");
    }
  };

  return (
    <form onSubmit={handleLogin}>
      <input
        type="email"
        placeholder="Email"
        value={email()}
        onInput={(e) => setEmail(e.currentTarget.value)}
        required
      />
      <input
        type="password"
        placeholder="Password"
        value={password()}
        onInput={(e) => setPassword(e.currentTarget.value)}
        required
      />
      <button type="submit">Login</button>
    </form>
  );
}
```
## Adding Navigation
SolidStart uses a file-based routing system, so all the files in your `src/routes` directory are automatically routes. To set up navigation:

1. Use the `<FileRoutes />` component from SolidStart.
2. Wrap it with `<Router>`from `@solidjs/router`.
3. Enclose everything in `<AuthProvider>` for app-wide authentication context.

Your App component should look like this:
```jsx
import { Router } from "@solidjs/router";
import { AuthProvider } from "./context/AuthContext";
import { Suspense } from "solid-js";
import { FileRoutes } from "@solidjs/start/router";

export default function App() {
  return (
    <AuthProvider>
      <Router root={props => <Suspense>{props.children}</Suspense>}>
        <FileRoutes />
      </Router>
    </AuthProvider>
  );
}
```
This setup enables automatic routing based on your file structure while providing authentication context throughout the app.

## Creating Job Listings Components
To use the `getDirectusInstance` to get data from Directus, create a `components` folder, inside the components folder create a `JobList.tsx` file and add the following:

```jsx
import { For, Show } from "solid-js";
import { Jobs, Job } from "../types";

interface JobListProps {
  jobs: Jobs;
  onEdit?: (job: Job) => void;
  onDelete?: (id: number) => void;
  onApply?: (id: number) => void;
}

function JobList(props: JobListProps) {
  return (
    <div class="container">
      <h2 class="title">Job Listings</h2>
      <ul class="job-list">
        <For each={props.jobs}>{(job: Job) => 
          <li class="job-list-item">
            <h3 class="job-title">{job.title}</h3>
            <p class="job-description">{job.description}</p>
            <p class="job-location">Location: {job.location}</p>
            <p class="job-type">Type: {job.type}</p>
            <p class="job-salary">Salary: ${job.salary}</p>
            <Show when={props.onEdit}>
              <button onClick={() => props.onEdit!(job)}>Edit</button>
            </Show>
            <Show when={props.onDelete}>
              <button onClick={() => props.onDelete!(job.id as number as number)}>Delete</button>
            </Show>
            <Show when={props.onApply}>
              <button onClick={() => props.onApply!(job.id as number)}>Apply</button>
            </Show>
          </li>
        }</For>
      </ul>
    </div>
  );
}

export default JobList;
```
The JobList component takes four props:
- `jobs`: An array of job objects to display
- `onEdit`: A function to handle job editing
- `onDelete`: A function to handle job deletion
- `onApply`: A function to handle job applications

These props allow the component to display jobs and respond to user actions for editing, deleting, and applying to jobs.

In the `routes/index.tsx` file use **JobList** component to display the job listings with the code:

```jsx
import { createResource, Show } from "solid-js";
import { Jobs } from "../types";
import { useAuth } from "../context/AuthContext";
import { readItems } from "@directus/sdk";
import getDirectusInstance from "~/lib/directus";
import { useNavigate } from "@solidjs/router";
import JobList from "~/components/JobList";

function HomePage() {
  const directus = getDirectusInstance();
  const auth = useAuth();
  const navigate = useNavigate();

  const fetchJobs = async () => {
    try {
      const fetchedJobs = await directus.request(readItems("job"));
      return fetchedJobs as Jobs;
    } catch (error) {
      console.error("Error fetching jobs:", error);
    }
  };

  const [jobs, { refetch: refetchJobs }] = createResource(fetchJobs);

  return (
    <div>
      <h1>Job Management System</h1>
      <Show
        when={auth.user()}
        fallback={
          <nav>
            <button onClick={() => navigate("/login")}>Login</button>
            <button onClick={() => navigate("/register")}>Register</button>
          </nav>
        }
      >
        <button onClick={auth.logout}>Logout</button>
        <Show when={auth.user()?.email === "admin@example.com"}>
          <button onClick={() => navigate("/applications")}>
            Manage Applications
          </button>
        </Show>
      </Show>
      <Show when={jobs.loading}>Loading jobs...</Show>
      <Show when={jobs.error}>Error loading jobs: {jobs.error}</Show>
    <Show
        when={!jobs.error}
        fallback={<div>Error loading jobs: {jobs.error?.message}</div>}
      >
        <JobList
          jobs={jobs() || []}
        />
      </Show>
    </div>
  );
}

export default HomePage;
```
![Job Listing Portal](<Screenshot 2024-07-02 at 10.43.05.png>)

## Creating, updating, and deleting job listings
Add the following functions to the `HomePage` component:
- `addJob`- Allows admins to list new jobs
- `updateJob`: Allows admins to edit job listings
- `deleteJob`: Enables admins to remove job listings

These functions will manage the respective actions when triggered by user interactions in the job list. Your `routes/index.tsx` file should have this updated code:

```jsx
import { createSignal, createResource, Show } from "solid-js";
import JobList from "~/components/JobList";
import JobForm from "~/components/JobForm";
import { Job, Jobs } from "../types";
import Modal from "~/components/Modal";
import { useAuth } from "../context/AuthContext";
import {
  readItems,
  createItem,
  updateItem,
  deleteItem,
} from "@directus/sdk";
import getDirectusInstance from "~/lib/directus";
import { useNavigate } from "@solidjs/router";

function HomePage() {
  const  directus  = getDirectusInstance();
  const [editingJob, setEditingJob] = createSignal<Job | null>(null);
  const [isModalOpen, setIsModalOpen] = createSignal(false);
  const [modalContent, setModalContent] = createSignal<"jobForm">("jobForm");
  const auth = useAuth();
  const navigate = useNavigate();

  const fetchJobs = async () => {
    try {
      const fetchedJobs = await directus.request(readItems("job"));
      return fetchedJobs as Jobs;
    } catch (error) {
      console.error("Error fetching jobs:", error);
    }
  };

  const [jobs, { refetch: refetchJobs }] = createResource(fetchJobs);

  const addJob = async (job: Omit<Job, "id">) => {
    try {
      if (!auth.user()) {
        throw new Error("You need to login to create a job");
      }
      job.employerId = auth.user()?.id as unknown as string;
      const response = await directus.request(createItem("job", job));

      if (response) {
        setIsModalOpen(false);
        refetchJobs();
      } else {
        throw new Error("Failed to add job");
      }
    } catch (error) {
      console.error("Error adding job:", error);
      alert("Failed to add job. Please try again.");
    }
  };

  const updateJob = async (updatedJob: Job, id: string) => {
    try {
      if (!auth.user()) {
        throw new Error("You need to login to create a job");
      }
      await directus.request(updateItem("job", id, updatedJob));
      setEditingJob(null);
      setIsModalOpen(false);
      refetchJobs();
    } catch (error) {
      console.error("Error updating job:", error);
      alert("Failed to update job. Please try again.");
    }
  };

  const deleteJob = async (id: number) => {
    try {
      await directus.request(deleteItem("job", id));
      refetchJobs();
    } catch (error) {
      console.error("Error deleting job:", error);
      alert("Failed to delete job. Please try again.");
    }
  };

  const openModal = (job?: Job) => {
    setEditingJob(job || null);
    setIsModalOpen(true);
  };

  return (
    <div>
      <h1>Job Management System</h1>
      <Show
        when={auth.user()}
        fallback={
          <nav>
            <button onClick={() => navigate("/login")}>Login</button>
            <button onClick={() => navigate("/register")}>Register</button>
          </nav>
        }
      >
        <button onClick={auth.logout}>Logout</button>
        <Show when={auth.user()?.email === "admin@example.com"}>
        <button
          onClick={() => {
            setModalContent("jobForm");
            setIsModalOpen(true);
          }}
        >
          Add New Job
        </button>
        <button onClick={() => navigate("/applications")}>Manage Applications</button>
        </Show>
      </Show>
      <Show when={jobs.loading}>Loading jobs...</Show>
      <Show when={jobs.error}>Error loading jobs: {jobs.error}</Show>
      <Show
        when={!jobs.error}
        fallback={<div>Error loading jobs: {jobs.error?.message}</div>}
      >
        <JobList
          jobs={jobs() || []}
          onEdit={auth.user()?.email === "admin@example.com"? openModal : undefined}
          onDelete={auth.user()?.email === "admin@example.com" ? deleteJob : undefined}
        />
      </Show>
      <Modal isOpen={isModalOpen()} onClose={() => setIsModalOpen(false)}>
        <Show when={modalContent() === "jobForm"}>
          <JobForm
            onSubmit={editingJob() ? updateJob : addJob}
            job={editingJob() as Job}
          />
        </Show>
      </Modal>
    </div>
  );
}

export default HomePage;
```
In the `components` folder, create two new files for the `JobForm.tsx` and `Modal.tsx` components that were used in your `HomePage` component. Add the following code to your `components/JobForm.tsx` file: 

```jsx
import { createSignal } from "solid-js";
import { Job } from "../types";

interface JobFormProps {
  job?: Job;
  onSubmit: (job: Job) => void;
}

export default function JobForm(props: JobFormProps) {
  const [title, setTitle] = createSignal(props.job?.title || "");
  const [description, setDescription] = createSignal(props.job?.description || "");
  const [location, setLocation] = createSignal(props.job?.location || "");
  const [type, setType] = createSignal(props.job?.type || "Full-time");
  const [salary, setSalary] = createSignal(props.job?.salary || 0);

  const handleSubmit = (e: Event) => {
    e.preventDefault();
    props.onSubmit({
        id: props.job?.id,
        title: title(),
        description: description(),
        location: location(),
        type: type(),
        salary: salary(),
    });
  };

  return (
    <form onSubmit={handleSubmit} class="job-form">
      <h2>{props.job ? "Edit Job" : "Add New Job"}</h2>
      <input
        type="text"
        placeholder="Job Title"
        value={title()}
        onInput={(e) => setTitle(e.currentTarget.value)}
        required
      />
      <textarea
        placeholder="Job Description"
        value={description()}
        onInput={(e) => setDescription(e.currentTarget.value)}
        required
      />
      <input
        type="text"
        placeholder="Location"
        value={location()}
        onInput={(e) => setLocation(e.currentTarget.value)}
        required
      />
      <select value={type()} onChange={(e) => setType(e.currentTarget.value)}>
        <option value="Full-time">Full-time</option>
        <option value="Part-time">Part-time</option>
        <option value="Contract">Contract</option>
      </select>
      <input
        type="number"
        placeholder="Salary"
        value={salary()}
        onInput={(e) => setSalary(Number(e.currentTarget.value))}
        required
      />
      <button type="submit">{props.job ? "Update Job" : "Add Job"}</button>
    </form>
  );
}
```
Then add the following code in your `components/Modal.tsx` file.

```jsx
import { Show, JSX } from "solid-js";
import "./Modal.css";

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: JSX.Element;
}

export default function Modal(props: ModalProps) {
  return (
    <Show when={props.isOpen}>
      <div class="modal-overlay" onClick={props.onClose}>
        <div class="modal-content" onClick={(e) => e.stopPropagation()}>
          <button class="modal-close" onClick={props.onClose}>×</button>
          {props.children}
        </div>
      </div>
    </Show>
  );
}
```
Create a new file named `Modal.css` in your **components** and copy the CSS styles [here](https://github.com/preshenv/directus-solidstart-job-board/blob/main/src/components/Modal.css)to it.

Log in with admin credentials, and click on the **Add New Job** button to create a new job, you can also edit and delete a job by clicking on the edit and delete buttons respectively

![Add new job modal](<Screenshot 2024-07-02 at 12.01.27.png>)

## Implementing search and filtering functionality
To implement job searching and filtering functionalities, update the code in your `components/JobList.tsx` file with the following:

```jsx
import { For, Show, createMemo, createSignal } from "solid-js";
import { Jobs, Job } from "../types";

interface JobListProps {
  jobs: Jobs;
  onEdit?: (job: Job) => void;
  onDelete?: (id: number) => void;
  onApply?: (id: number) => void;
}

function JobList(props: JobListProps) {
  const [searchQuery, setSearchQuery] = createSignal("");
  const [jobType, setJobType] = createSignal("All");
  const [minSalary, setMinSalary] = createSignal(0);
  const [maxSalary, setMaxSalary] = createSignal(1000000);

  const filteredJobs = createMemo(() => {
    const query = searchQuery().toLowerCase();
    return props.jobs.filter(
      (job: Job) =>
        (job.title.toLowerCase().includes(query) ||
          job.description.toLowerCase().includes(query) ||
          job.location.toLowerCase().includes(query)) &&
        (jobType() === "All" || job.type === jobType()) &&
        job.salary >= minSalary() &&
        job.salary <= maxSalary()
    );
  });

  return (
    <div class="container">
      <input
        type="text"
        class="search-input"
        placeholder="Search jobs..."
        onInput={(e) => setSearchQuery(e.currentTarget.value)}
        value={searchQuery()}
      />
      <div class="filters">
        <select onChange={(e) => setJobType(e.currentTarget.value)}>
          <option value="All">All Types</option>
          <option value="Full-time">Full-time</option>
          <option value="Part-time">Part-time</option>
          <option value="Contract">Contract</option>
        </select>
        <input
          type="number"
          placeholder="Min Salary"
          onInput={(e) => setMinSalary(parseInt(e.currentTarget.value) || 0)}
        />
        <input
          type="number"
          placeholder="Max Salary"
          onInput={(e) => setMaxSalary(parseInt(e.currentTarget.value) || 1000000)}
        />
      </div>
      <ul class="job-list">
        <For each={filteredJobs()}>
          {(job: Job) => (
            <li class="job-list-item">
              <h3 class="job-title">{job.title}</h3>
              <p class="job-description">{job.description}</p>
              <p class="job-location">Location: {job.location}</p>
              <p class="job-type">Type: {job.type}</p>
              <p class="job-salary">Salary: ${job.salary}</p>
              <Show when={props.onEdit}>
                <button onClick={() => props.onEdit!(job)}>Edit</button>
              </Show>
              <Show when={props.onDelete}>
                <button onClick={() => props.onDelete!(job.id as number)}>
                  Delete
                </button>
              </Show>
              <Show when={props.onApply}>
                <button onClick={() => props.onApply!(job.id as number)}>
                  Apply
                </button>
              </Show>
            </li>
          )}
        </For>
      </ul>
    </div>
  );
}

export default JobList;
```
The update implements the following features:

- Search functionality: Users can search jobs by title, description, or location using the search input.
- Job type filtering: A dropdown allows users to filter jobs by type (Full-time, Part-time, Contract, or All).
- Salary range filtering: Users can set minimum and maximum salary ranges.
- Reactive filtering: The `createMemo` function creates a reactive filtered job list based on the search query and filter criteria.
![Job listing with search and filter](<Screenshot 2024-07-02 at 11.55.27.png>)

## Implementing job application functionality
Create a new file named `ResumeForm.tsx` in your components folder and add the code snippets below for the resume url inputs.

```jsx
import { createSignal } from "solid-js";

interface ResumeFormProps {
  onSubmit: (resumeUrl: string) => void;
}

function ResumeForm({ onSubmit }: ResumeFormProps) {
  const [resumeUrl, setResumeUrl] = createSignal("");

  const handleSubmit = (e: Event) => {
    e.preventDefault();
    onSubmit(resumeUrl());
  };

  return (
    <form onSubmit={handleSubmit}>
      <h2>Submit Your Resume</h2>
      <div>
        <label for="resumeUrl">Resume URL:</label>
        <input
          type="url"
          id="resumeUrl"
          value={resumeUrl()}
          onInput={(e) => setResumeUrl(e.currentTarget.value)}
          required
        />
      </div>
      <button type="submit">Submit</button>
    </form>
  );
}

export default ResumeForm;
```
Update the code in your `src/routes/index.tsx` file to add the job application functionality with the following code.

```jsx
+
//...  //...(your existing imports
import ResumeForm from "~/components/ResumeForm";

function HomePage() {
     //...(your existing state varribles
    const [modalContent, setModalContent] = createSignal<"jobForm" | "resumeForm">("jobForm");
    const [applyingJobId, setApplyingJobId] = createSignal<number | null>(null);

     // ... (your existing code for fetchJobs, addJob, updateJob, deleteJob, and openModal)

    const applyForJob = async (jobId: number) => {
      if (!auth.user()) {
        alert("You need to login to apply for a job");
        return;
      }
      setApplyingJobId(jobId);
      setModalContent("resumeForm");
      setIsModalOpen(true);
    };

    const submitApplication = async (resumeUrl: string) => {
      try {
        if (!auth.user() || !applyingJobId()) {
          throw new Error("You need to login to apply for a job");
        }
        const newApplication = {
          job: applyingJobId(),
          user: auth.user()?.id as unknown as string,
          status: "pending",
          resumeUrl: resumeUrl,
        };

        await directus.request(createItem("application", newApplication));
        alert("Application submitted successfully!");
        setIsModalOpen(false);
        setApplyingJobId(null);
      } catch (error) {
        alert("Failed to apply for job. Please try again.");
      }
    };

    return (
      <div>
        <h1>Job Portal</h1>
        {/* ... (your existing code for authentication buttons) */}
        <Show when={jobs.loading}>Loading jobs...</Show>
        <Show when={jobs.error}>Error loading jobs: {jobs.error}</Show>
        <Show
          when={!jobs.error}
          fallback={<div>Error loading jobs: {jobs.error?.message}</div>}
        >
          <JobList
            jobs={jobs() || []}
            onEdit={auth.user()?.email === "admin@example.com" ? openModal : undefined}
            onDelete={auth.user()?.email === "admin@example.com" ? deleteJob : undefined}
            onApply={auth.user()?.email !== "admin@example.com" ? applyForJob : undefined}
          />
        </Show>
        <Modal isOpen={isModalOpen()} onClose={() => setIsModalOpen(false)}>
          <Show when={modalContent() === "jobForm"}>
            <JobForm
              onSubmit={editingJob() ? updateJob : addJob}
              job={editingJob() as Job}
            />
          </Show>
          <Show when={modalContent() === "resumeForm"}>
            <ResumeForm onSubmit={submitApplication} />
          </Show>
        </Modal>
      </div>
    );
  }

export default HomePage;
```
Register as an applicant, click on the Apply button to show the resume URL modal, enter a resume URL, and click on Submit to apply for a job.
![job application functionality](<Screenshot 2024-07-02 at 13.07.04.png>)

## Managing applicant profiles and resumes
To allow the admin to view, accept, or decline job applications, create a new file named `applications.tsx` in your `src/routes` folder and add the following code:

```jsx
import { createSignal, createEffect, For, Show } from "solid-js";
import { readItems, updateItem } from "@directus/sdk";
import { useAuth } from "../context/AuthContext";
import { Application, Job } from "../types";
import getDirectusInstance from "~/lib/directus";

const ManageApplicationsPage = () => {
  const directus  = getDirectusInstance();

  const [applications, setApplications] = createSignal<Application[]>([]);
  const [jobs, setJobs] = createSignal<Job[]>([]);
  const [selectedApplication, setSelectedApplication] =
    createSignal<Application | null>(null);
  const auth = useAuth();

  const fetchApplications = async () => {
    try {
      const fetchedApplications = await directus.request(
        readItems("application", {
          sort: ["-date_created"],
          deep: {
            userId: {
              fields: ["first_name", "last_name"],
            },
            jobId: {
              fields: ["title"],
            },
          },
          fields: ["*", "userId.first_name", "userId.last_name", "jobId.title"],
        })
      );
      setApplications(fetchedApplications as Application[]);
    } catch (error) {
      console.error("Error fetching applications:", error);
    }
  };

  const fetchJobs = async () => {
    try {
      const fetchedJobs = await directus.request(readItems("job"));
      setJobs(fetchedJobs as Job[]);
    } catch (error) {
      console.error("Error fetching jobs:", error);
    }
  };

  createEffect(() => {
    fetchApplications();
    fetchJobs();
  });

  const updateApplicationStatus = async (id: number, status: string) => {
    try {
      await directus.request(updateItem("application", id, { status }));
      fetchApplications();
    } catch (error) {
      console.error("Error updating application status:", error);
    }
  };

  return (
    <div class="container mx-auto p-4">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div>
          <h2 class="text-xl font-semibold mb-2">Applications List</h2>
          <For each={applications()}>
            {(application) => (
              <div
                class="border p-2 mb-2 cursor-pointer hover:bg-gray-100"
                onClick={() => setSelectedApplication(application)}
              >
                <p>Job: {application.job.title}</p>
                <p>Applicant: {application.user.first_name} {application.user.last_name}</p>
                <p>Status: {application.status}</p>
              </div>
            )}
          </For>
        </div>
        <Show when={selectedApplication()}>
          <div class="border p-4">
            <h2 class="text-xl font-semibold mb-2">Application Details</h2>
            <p>Job: {selectedApplication()?.job.title}</p>
            <p>Applicant: {selectedApplication()?.user.first_name} {selectedApplication()?.user.last_name}</p>
            <p>Status: {selectedApplication()?.status}</p>
            <p>
              Resume:
              <a
                href={selectedApplication()?.resumeUrl}
                target="_blank"
                rel="noopener noreferrer"
              >
                View Resume
              </a>
            </p>
            <div class="mt-4">
              <button
                class="bg-green-500 text-white px-4 py-2 mr-2"
                onClick={() =>
                  updateApplicationStatus(selectedApplication()!.id, "accepted")
                }
              >
                Accept
              </button>
              <button
                class="bg-red-500 text-white px-4 py-2"
                onClick={() =>
                  updateApplicationStatus(selectedApplication()!.id, "rejected")
                }
              >
                Reject
              </button>
            </div>
          </div>
        </Show>
      </div>
    </div>
  );
};

export default ManageApplicationsPage;
```
Here we implemented the following:

- `fetchApplications()`: Retrieves all job applications from the Directus backend, including related user and job information.
- `fetchJobs()`: Fetches all available jobs from the Directus backend.
- `updateApplicationStatus()`: Updates the status of a job application (accepted or rejected) in the Directus backend.
- `ManageApplicationsPage` component: Renders the application management interface, including:
  - A list of all applications
  - Detailed view of a selected application
  - Buttons to accept or reject the selected application

Click on the **Mananage Applications** button to navigate to the application's route.

![Job applications management](<Screenshot 2024-07-02 at 12.26.15.png>)


## Summary
In this tutorial, you’ve learned how to build a job portal with Directus and SolidStart.js, dynamically create, read, update, and delete jobs and applications, and successfully build a job portal application with Directus for the backend and SolidStart.js for the frontend.
Explore the Directus documentation to discover other amazing features you can add to your SolidStart.js applications.

