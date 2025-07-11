American Refining Group Documentation: AS400 IBM i (ERP)

Welcome to the documentation repository for the ARG-AS400 legacy application, built using MkDocs and hosted on GitHub Pages.
Overview
This repository contains the source files for the documentation of the ARG-AS400 application, generated using MkDocs with the Material theme. The site is automatically deployed to GitHub Pages whenever changes are pushed to the main branch.
You can view the live documentation at: https://american-refining-group.github.io/DOCS-AS400-IBMi/
Prerequisites
To contribute to this repository, ensure you have the following installed:

Git: To clone and manage the repository.
Python (3.6 or higher): Required for MkDocs.
MkDocs and dependencies: Install via requirements.txt (see below).

How to Pull and Contribute
1. Clone the Repository
Clone the repository to your local machine:
git clone https://github.com/American-Refining-Group/DOCS-AS400-IBMi.git
cd DOCS-AS400-IBMi

2. Set Up the Environment
Install the required Python dependencies listed in requirements.txt:
pip install -r requirements.txt

3. Preview Changes Locally
To preview the documentation site locally:
mkdocs serve

Open your browser to http://localhost:8000 to view the site. Make sure it renders correctly before proceeding.
4. Make Changes

Edit the Markdown files in the docs/ directory to update the documentation.
If you need to modify the site configuration, edit mkdocs.yml.
Test your changes locally with mkdocs serve to ensure they render as expected.

5. Commit and Push Changes
Once you're satisfied with your changes:
git add .
git commit -m "Describe your changes here"
git push origin main

Note: You must have write access to the repository to push directly to main. If you don’t, fork the repository, make changes in your fork, and submit a pull request to repo
6. Deployment

A GitHub Actions workflow (.github/workflows/deploy.yml) automatically builds and deploys the site to the gh-pages branch when you push to main.
The deployment process takes about 1-2 minutes to complete.
After the workflow finishes, the updated documentation will be live at {TBD}.

You can monitor the workflow status in the Actions tab of the repository.
Troubleshooting

Workflow fails: Check the Actions tab for error logs. Common issues include missing dependencies in requirements.txt or permission errors.
Site not updating: Ensure the gh-pages branch is set as the GitHub Pages source in Settings > Pages. If the site isn’t live, verify the workflow completed successfully.
Local preview issues: Run pip install -r requirements.txt again and ensure all dependencies (e.g., mkdocs, mkdocs-material, mkdocs-awesome-pages-plugin) are installed.

Contributing Guidelines

Keep changes focused and well-documented in your commit messages.
Test changes locally before pushing.
If adding new features or plugins to mkdocs.yml, update requirements.txt with any new dependencies.
For major changes, consider opening an issue or discussing with the maintainers first.

Contact
For questions or issues, open an issue in the repository or contact the maintainers.