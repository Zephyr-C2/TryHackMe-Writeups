# Crack the Hash Level 2

I will be skipping the first four tasks as they are already guided. 

# Tasks

### Task 5 (Custom Wordlist Generation):

At this point you should have wordlistctl set up. We will need it for the rest of the challenge. 

Unfortunately [Mentalist](https://github.com/sc0tfree/Mentalist) is a bit outdated, which comes with some issues if using a newer version of python. There's two ways to solve this. I'm going to be explaining the 'proper' way using pyenv. Otherwise you can modify the code manually as discussed in this [issue](https://github.com/sc0tfree/Mentalist/issues/41) or look for a working fork.

#### pyenv method

This is going to be a quick overview of how to use [pyenv](https://github.com/pyenv/pyenv#set-up-your-shell-environment-for-pyenv), its installation and use may differ slightly depending on which shell you're using. I recommend following the full instructions on the github page.

* Automatic installer 
  
  ```
  curl -fsSL https://pyenv.run | bash
  ```

* Set up shell enviroment for Pyenv (follow instructions on github for your corresponding shell, most likely bash)

* Create a directory for Mentalist enviroment
  
  ```
  mkdir Mentalist-env
  ```

* Clone Mentalist
  
  ```
  git clone https://github.com/sc0tfree/Mentalist
  ```

* Move into the Mentalist directory
  
  ```
  cd Mentalist
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

* Install Mentalist (make sure you're in the right directory if you changed to test pyenv)
  
  ```
  pip install .
  ```
  
  If you did everything correctly it should say 'Successfuly installed Mentalist-1.0'.

* Run Mentalist (I used sudo because I was getting permission errors when trying to save the wordlist)
  
  ```
  sudo Mentalist
  ```
  
  You should see the Mentalist GUI open. While that was slightly annoying, it's important to understand how to run things that rely on deprecated versions of python.

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
* Now open Mentalist
* Remove the preloaded English dictionary wordlist, add the dog wordlist.
* Press the plus in the top right, select Case, then click the plus next to Case. Select `Toggle Nth...` and then enter 2
* Press the plus in the top right, select Substitution, then click the plus next to Substitution. Select `Replace All Instances...` and then select `S -> $`

| ![mentalist_wordlist](https://github.com/user-attachments/assets/659fb317-4531-40f3-832f-634a397f5a3f) |
|:------------------------------------------------------------------------------------------------------:|
| *It should look like this.*                                                                            |

* Click Process, select Full Wordlist, give the new wordlist a name and save it

##### Using the dog wordlist with JohnTheRipper

* Make a file with the given hash
  
  ```
  echo "ed91365105bba79fdab20c376d83d752" > md5.txt
  ```
* Run JohnTheRipper with the crafted wordlist
  
  ```
  john --format=raw-MD5 --wordlist=/usr/share/wordlists/misc/mentalist_dog_list.txt md5.txt
  ```
  
  | ![dog_answer](https://github.com/user-attachments/assets/613654cb-a1e2-48cd-8d6d-7d91cb3b9153) |
  |:----------------------------------------------------------------------------------------------:|
  | *You should get the answer.*                                                                   |

#### Using ceWL
* Install [ceWL](https://github.com/digininja/CeWL) by following the instructions on the github page
* Go to the directory you want to work in
* Run the given command
	```
	cewl -d 2 -w $(pwd)/example.txt https://example.org
	```
* Test if it worked by running cat on example.txt
	```
	cat example.txt
	```
	| ![](../../Medium_Writeups/Crack_the_Hash_2/cewl_answer.png) |
	| :-: |
	| *Something went wrong during this part for me, the answer is information* |
I'm not sure if I did something wrong, or if it's broken, but the list is not in order for me. I brute forced the answer.

#### Using TTPassgen 

* TTPassgen can be easily installed using pip:
	```
	pip install TTPassgen
	```
	In my case I actually needed to use pipx, your mileage may vary.

* Create the first wordlist containing all 4 digit PIN code values.
	```
	ttpassgen --rule '[?d]{4:4:*}' pin.txt
	```

* Create the second list containing all lowercase char combinations of 1-3 length.
	```
	ttpassgen --rule '[?l]{1:3:*}' abc.txt
	```

* Combine the two wordlists.
	```
	ttpassgen --dictlist 'pin.txt,abc.txt' --rule '$0[-]{1}$1' combination.txt
	```

* Now let's crack the given hash using John and the crafted wordlist.
	```
	echo e5b47b7e8df2597077e703c76ee86aee > MD5.txt
	```
	```
	john --format=raw-MD5 --wordlist=/path/to/wordlist MD5.txt
	```
	| ![](../../Medium_Writeups/Crack_the_Hash_2/ttpass_answer.png) |
	| :-: |
	| *Answer output looks like this*|

### TASK 6 (It's time to crack hashes)
* Launch the machine and connect via VPN or Attack Box.

* Go to the machine's IP and select 'advices' in the top right
	| ![](../../Medium_Writeups/Crack_the_Hash_2/advice_1.png) |
	| :-: | 
	| *The first challenge* | 
The user says their name is John, and they usually use their son's name. The hacker recommends using a border mutation at the beginning, end, or both.

* Let's grab a male name wordlist from wordlistctl. I'm running the wordli 
	```
	./wordlistctl.py search male
	```
	![](../../Medium_Writeups/Crack_the_Hash_2/male_wordlist.png)
* Let's try option number 2. The option -d will automatically decompress the archive.
	```
	sudo ./wordlistctl fetch -d Malename
	```

* Now open mentalist.
* Add the wordlist we just downloaded like we did with the dog one earlier.


* Check the hash type with haiti (explained earlier in the room, skipped in this walkthrough)
	```
	haiti b16f211a8ad7f97778e5006c7cecdf31
	```
	![](../../Medium_Writeups/Crack_the_Hash_2/hash1_type.png)

* We see that it's an MD5.
 	```
 	echo b16f211a8ad7f97778e5006c7cecdf31 > MD5.txt
 	```
 
