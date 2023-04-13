# JenkinDemo

## Steps for setting up history for jenkin history

1. create the jenkin pipeline
2. go to Pipeline panel and set up like mentioned below
<img width="968" alt="Screenshot 2023-04-13 at 3 58 31 PM" src="https://user-images.githubusercontent.com/89962651/231732338-6454b9f1-e3b0-4f75-81ce-3eac67db1e1d.png">
<img width="968" alt="Screenshot 2023-04-13 at 3 58 52 PM" src="https://user-images.githubusercontent.com/89962651/231732435-b999468a-ac0f-4f2e-a054-17222e2cf692.png">
<img width="968" alt="Screenshot 2023-04-13 at 3 59 24 PM" src="https://user-images.githubusercontent.com/89962651/231732366-ec90d4c3-5a13-4d48-8901-543507c73975.png">

Parameter Information:
1. SCM(Source Control Manager) -> spcify it Git, Gitlab or any tool of your choice
2. Repository URL:- need to add git repo url where it have our jenkin file
3. Credentials :- it can be set up using many ways but I explore two method.
5. Branch Specifier (blank for 'any') :- current branch name
6. Repository browser :- should be Auto for our case
7. Script Path :- name of the jenkins file in the repository

## Credentials
there are many ways to set up credentials

### Username with Password
<img width="912" alt="Screenshot 2023-04-13 at 4 10 31 PM" src="https://user-images.githubusercontent.com/89962651/231734677-bdce5b8c-1eb6-4016-b1d7-22388cf067fd.png">
<img width="912" alt="Screenshot 2023-04-13 at 4 10 43 PM" src="https://user-images.githubusercontent.com/89962651/231734688-fec2a494-a1ae-4388-b047-441e533d7a7a.png">
