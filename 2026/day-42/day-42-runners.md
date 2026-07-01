# Day 42 – Runners: GitHub-Hosted & Self-Hosted

## Task
Every job needs a machine to run on. Today we will understand **runners** — GitHub's hosted ones and how to set up our own self-hosted runner on a real server.

---

## GitHub-Hosted Runners

1. Create a workflow with 3 jobs, each on a different OS:
   - `ubuntu-latest`
   - `windows-latest`
   - `macos-latest`
2. In each job, print:
   - The OS name
   - The runner's hostname
   - The current user running the job
3. Watch all 3 run in parallel


![Job on Ubuntu](On%20Ubuntu.png)
![Job on macos](On%20macos.png)
![Job on windows](On%20windows.png)


### What is a GitHub-hosted runner? 

- A GitHub-hosted runner is a virtual machine or container used to execute jobs in a GitHub Actions workflow. It provides a clean, pre-configured environment (with operating systems like Linux, Windows, or macOS) to automatically build, test, and deploy code.

- They come pre-installed with various tools, SDKs, and software packages.


### Who manages it?

These runners are fully provided and managed by GitHub . When you use one, you do not need to maintain the underlying infrastructure or manually install operating system updates. GitHub takes care of:
   - Machine provisioning and scaling
   - Software updates and patching 
   - Hardware maintenance 

---

## Explore What's Pre-installed
1. On the `ubuntu-latest` runner, run a step that prints:
   - Docker version
   - Python version
   - Node version
   - Git version
2. Look up the GitHub docs for the full list of pre-installed software on `ubuntu-latest`

![Checking we get pre-installed softwares or not](Check%20Pre-Installed%20Softwares.png)


### Why does it matter that runners come with tools pre-installed?

Runners with pre-installed tools dramatically speed up your CI/CD pipelines. Instead of spending time downloading SDKs, databases, or runtime environments on every run, your jobs execute immediately .

The main reasons why this matters include:
- Instant Compilation & Execution
- No Network Dependencies
- Effortless Scalability 


---

## Set Up a Self-Hosted Runner

1. Go to your GitHub repo → Settings → Actions → Runners → **New self-hosted runner**
2. Choose Linux as the OS
3. Follow the instructions to download and configure the runner on:
   - Your local machine, OR
   - A cloud VM (EC2, Utho, or any VPS)
4. Start the runner — verify it shows as **Idle** in GitHub

**Verified :** Our runner appears in the Runners list with a green dot.

![Runner is Idle on github](Runner's%20Status.png)

---

## Use Your Self-Hosted Runner

1. Create `.github/workflows/self-hosted.yml`
2. Set `runs-on: self-hosted`
3. Add steps that:
   - Print the hostname of the machine (it should be YOUR machine/VM)
   - Print the working directory
   - Create a file and verify it exists on your machine after the run
4. Trigger it and watch it run on your own hardware


![Job completed by self-hosted runner](job%20completed%20verified.png)
![File created as task given in job](file%20created%20verified.png)


---

## Labels

1. Add a **label** to your self-hosted runner (e.g., `my-linux-runner`)
2. Update your workflow to use `runs-on: [self-hosted, my-linux-runner]`
3. Trigger it — does it still pick up the job?

![Job Completed with Custom Label](Self-Hosted%20Runner%20with%20Custome%20Label.png)



### Why are labels useful when you have multiple self-hosted runners?

Customn Labels for Self-Hosted runners are essential for routing workflow jobs to the correct machines when managing multiple self-hosted runners. They give you the flexibility to manage diverse computing environments and prevent jobs from executing on the wrong infrastructure.