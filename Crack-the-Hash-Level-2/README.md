# Crack the Hash Level 2 
I will be skipping the first four tasks as they are already guided. 

# Tasks

### Task 5 (Custom Wordlist Generation):
At this point you should have wordlistctl set up. We will need it for the rest of the challenge. 

Unfortunately [Mentalist](https://github.com/sc0tfree/mentalist) is a bit outdated, which comes with some issues if using a newer version of python. There's two ways to solve this. I'm going to be explaining the 'proper' way using pyenv. Otherwise you can modify the code manually as discussed in this [issue](https://github.com/sc0tfree/mentalist/issues/41) or look for a working fork.

#### pyenv method 
This is going to be a quick overview of how to use [pyenv](https://github.com/pyenv/pyenv#set-up-your-shell-environment-for-pyenv), its installation and use may differ slightly depending on which shell you're using. I recommend following the full instructions on the github page.

* Automatic installer 
```
curl -fsSL https://pyenv.run | bash
```
* Set up shell enviroment for Pyenv (follow instructions on github for your corresponding shell, most likely bash)

* Create a directory for mentalist enviroment
```
mkdir mentalist-env
```
* Clone mentalist
```
git clone https://github.com/sc0tfree/mentalist
```
* Move into the mentalist directory
```
cd mentalist
```
* Install correct python version using pyenv
```
pyenv install 3.11.8
```
* Create local python enviroment using pyenv
```
pyenv local 3.11.8
```
* Test python version
```
python --version
```
* You should see this as a response
```
Python 3.11.8
```
If you move into a different directory and check version again, it should be your global version, not 3.11.8. 
* Install mentalist (make sure you're in the right directory if you changed to test pyenv)
```
pip install .
```
If you did everything correctly it should say 'Successfuly installed Mentalist-1.0'.
* Run mentalist
```
mentalist
```
You should see the mentalist GUI open. While that was slightly annoying, it's important to understand how to run things that rely on deprecated versions of python.

Now we can move on to actually crafting the wordlist.

#### Crafting dog wordlist 
* First fetch (haha) the dog wordlist using wordlistctl. I did this by navigating to where I had the script installed and running:
```
sudo ./wordlistctl.py fetch dogs
```
This will download a few folders in your /usr/share/wordlists directory. We are looking for the /usr/share/wordlists/misc/dogs.txt.gz
* Unzip it
```
gunzip dogs.txt.gz
```
* Now open mentalist
* Remove the preloaded English dictionary wordlist, add the dog wordlist.
* Press the plus in the top right, select Case, then click the plus next to Case. Select `Toggle Nth...` and then enter 2.
* Press the plus in the top right, select Substitution, then click the plus next to Substitution. Select `Replace All Instances...` and then select `S -> $`
