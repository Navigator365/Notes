# Part 1

**Easy**
1. We'll begin by clicking on his contact info on his linkedin. This brings us to his https://github.com/morimori5/, where we see an @therealkeimori in his bio. Looking this up brings us to https://x.com/therealkeimori, where one tweet gives a link to his https://www.goodreads.com/user/show/198059115-kei-mori, and an older tweet states he's on universeondon as probablytherealkeimori, or https://universeodon.com/@probablytherealkeimori. Navigating to the Mastodon website, he's linked a reddit profile in his bio.  https://www.reddit.com/user/Extension_Strength15/ Finally, a tweet references running a marathon, and gives a number 71254748. This isn't a phone number, but is instead a link to his Strava profile, with the number being his id, at https://www.strava.com/athletes/71254748.
2. Going through his tweets, we can see he reposts a lot of Giants content, and comments on them favorably, so he's probably a Giants fan. 
3. From his github bio, we see his organization is set to Stony Brook University, whose tweets he occasionally reposts. As we know he's not working there currently thanks to his linkedin, he probably went to college there. 
4. He lives in New York City, as given by his location in his github bio, and by comments that he's attended NYC marathons, it's likely he lives in DC
5. His job title is given on his linkedin as Software Developer. 
6. On Linkedin, he's currently working at BuzzAI. 

**Medium**
1. We can clone a Github repository of his, and check commit messages to find what email is associated with his account. This gives us keimorimori5@gmail.com.
2. On linkedin, he's set his birthday as September 12, and in his goodreads bio, he gives his age as 36 with a join date of 2026 (meaning his age is current), so he was born on September 12, 1990. 
3. His phone number is given in his goodreads bio as well (requires clicking more at first):  (202) 838-6803‬‬
4. On his universeondon account, he's posted a picture with the Washington monument visible, saying he was there "last Monday", which corresponds to January 12, so he was in Washington, D.C.
5. Japan, this was obtained through a social engineering script given below. 
6. Looking at images he's posted on his twitter, we can see he posted a picture of rose fireworks. Clicking on it to see replies, we can see him implying it happened on January 1st, 2024 (the post date is January 13, 2024, and he says in a reply that he'll have to find another fun new year's activity for 2025). Reverse-image searching the photo, we find it's of the 2024 Rose Bowl, with the same image at another angle as verification here: https://skyelementsdrones.com/drone-events/rose-bowl-drone-show

**Hard**
1. Github stores public ssh keys of accounts at github.com/account_name.keys, so a key of his is at https://github.com/morimori5.keys
2. His first pet was a dog named Max, this was also obtained through social engineering. 
3. Looking at his strava account, we can see he commonly runs past the intersection of 3rd Avenue and East 89th street, which would be a good place to intercept him.  
4. His pin is 9172; this was also obtained through social engineering.
5. He was born in Pittsburgh; this was also obtained through social engineering. 
6. If he has a github, he might also have a gitlab. Searching for a gitlab with the same username he used for github, we find his gitlab account. He as a project called Wadsworth, https://gitlab.com/morimori5/wadsworth, and his status page showed he was last working on it (commenting on an issue) on December 2nd, 2020. 


**Challenging**
1. We can use Epieos to look up his google account for any associated accounts, and find a google maps profile with his name. Around the time in question, he left a review for a restaurant called the Little Pearl, suggesting this is where he went to dinner. 
2. For this, we'll need to use bitbucket's api to access the password_manager project referenced in the Wadsworth issue. Thanfully, the issue comment provides us with base64-encoded client ids and client secrets. After decoding these, we'll use documentation found here https://developer.atlassian.com/cloud/bitbucket/oauth-2/ to get an access token so that we can access the password manager. Do do this, we run the command ``curl -X POST https://bitbucket.org/site/oauth2/access_token      -u "C7vV2RRKQW9fTyANGv:vyZSf6Sz4fxkL3dLWsEwNpvCxVVfgE9k"      -d grant_type=client_credentials`` to get a token returned as json, and then use that token to clone the repository to our local machine via the command `$ git clone https://x-token-auth:{access_token}@bitbucket.org/user/repo.git`. This gives us the password_manager repo, which has a list of account passwords. Looking at the only file in it, we can see the password to TJMaxx is H0ldm3t!ght. 
3. Going through git logs  in the password_manager repo to see all passwords that were present at any time in the manager, we see that they're all based on Beatles songs, with a tendency for @s and 3s to be inserted for o's and e's, respectively. 
4. Using the email discovered previously along with his TJMaxx password, we can log into his TJMaxx account, click on Billing, and find the the info: XXXXXXXXXXXX8820 Expiring 12/2030
5. While in TJMaxx, we can click on Shipping and find his full address at 1623 3RD AVE, NEW YORK, NY 10128-3638. 

**Bonus**
2. Using the Maigret OSINT tool, we give the username  therealkeimori to find this pastebin: https://pastebin.com/TF7J9i1m indicating Hina is going to the grocery store to buy the following: Milk, Eggs, Broccoli, Apples, Bananas, Trader joe's pizza dough, Chicken
3. We'll Google reverse-image search the photo of the Washington monument he posted on universeondon. We find another similar-looking terrace with a view of the monument (associated with a now-deleted tweet from the @JMUdcsemester x account) and Google reverse-image search that. Among the similar images is a link to a tiktok page for the Roof Terrace Restaurant at the Kennedy Center  - https://www.tiktok.com/discover/roof-terrace-restaurant-dc-kennedy-center. Watching a few videos confirms this is the same view as in the photo.  As the universeondon post mentioned networking, we'll look for networking events near the Kennedy Center on the date in question via Google, and find this:  https://www.eventbrite.com/e/federal-government-contractors-winter-soiree-networking-event-2026-tickets-1899061216219, meaning he was at a federal contractor networking event. 
**Social Engineering Script**: 
Hello, is this Kei Mori?

Hello sir, This is Amazon Customer Service's Fraud Detection Divison. As part of our ongoing security processing, we've detected an Amazon account associated with your name and with password !s@wh3rst@nd!ngth3r3 has been accessed in the Donetsk People's Republic to place orders in excess of $1000. As your account information indicates you're residing in the United States, pursuant to the Code of Federal Regulations parts 730-774 and guidance from the Department of State's Office of Foreign Export Control, we have disabled your account and cancelled any outstanding orders placed on it. It can only be reactivated upon positive verification of your identity, after which we will perform a security reconfigurtaion to secure your account from any future attacks. Would you like to begin the identity verification process sir?


Very well sir. Could you begin by verifying your address with zip code 10128-3638?

SHOULD BE: 1623 3RD AVE, NEW YORK, NY 10128-3638

Thank you sir. Could you verify the last 4 digits of the Mastercard expiring December 2030?

SHOULD BE: 8820

Thank you sir, we've completed the identity verification process. As part of Amazon's Fraud Prevention program, we collaborate with the financial institutions of victims of fraud to reverse any unauthorized charges. To connect our system to your financial institution, we require some information in order to reverse the charges. Could you please provide the name of the financial institution associated with the card on file with us?

Chase card

Thank you sir. <Pause, sound of mouse scrolling and typing> [Institution name] is a member of the Federal Deposit Insurance Corporation's Fraud Prevention Consortium, an organization of financial insitutions with a common fraud-prevention system. These institutions require third-party organizations like ours to provide a customer's ATM PIN, as issued by a member institution, for them to accept any of our reporting. Could you please provide your [institution name]-assigned ATM pin so that we can submit a fraud report to reverse any charges to your account?

ANSWER: 9172

Thank you sir. We'll now begin the security reconfiguration process. In this process, we'll set up new security questions associated with your account, to prevent any future unauthorized access of your account. You don't need to remember the questions, only the answers. Are you ready to begin?


Thank you sir. The first security question is: what was the name and type of your first pet?

ANSWER: Dog named Max

Thank you sir. The second question is: in what city were you born?

ANSWER: Pittsburgh

Thank you sir. The third security question is: where was your most recent international trip?

ANSWER: Japan 

Thank you sir, we've completed the security reconfiguration process to secure your account. Do you have any questions?

Thank you for taking this call, sir. Have a nice day.
# Part 2

1. This can all be answered with recon/hosts-hosts/resolve in recon-ng, with SOURCE being set to {subdomain}.umd.edu
	1. www.umd.edu => 165.227.248.71
	2. ftp.umd.edu => Unknown, so doesn't exist
	3. firewall.umd.edu => Unknown, so doesn't exist
	4. proxy.umd.edu => DNS Error, so doesn't exist
	5. router.umd.edu => 10.182.173.1
	6. admin.umd.edu => 23.185.0.253
	7. www2.chem.umd.edu => 129.2.148.78
	8. mx.umd.edu => 10.105.177.33
	9. pop.umd.edu => 128.8.10.197
	10. cash.umd.edu => Unknown, so doesn't exist
	11. rice.umd.edu => 129.2.248.107
	12. starfleet.umd.edu => 129.2.146.76
	13. blue.umd.edu => No answer, so doesn't exist
	14. green.umd.edu => 128.8.15.50
	15. 4yearplans.umd.edu => 52.85.31.120, 52.85.31.19, 52.85.31.80, 52.85.31.115
2. We'll use shodan
	1. We'll use the query product:"Microsoft-IIS" org:"University of Maryland" query in Shodan, then look at the top versions sidebar on the left, to find that 10.0 is the most common version. 
	2. Google tells us AFP runs on port 548, so we'll use port:548  org:"University of Massachusetts at Boston" as our search term in Shodan. This gives us a result that points to scass.fiske.umb.edu at IP 158.121.48.76, and is running AFP. For verification, we check the url and see it's a portal to the "Fiske Center Login". Looking this up, we see it's a center for archeological research, run by the UMB anthropology department, so this is the right server. 
	3. We'll use product:"serialnumberd" org:"Technical University of Kenya" as our Shodan query, giving the IP 41.89.56.20 running a Mac  OS X Server serialnumberd on UDP port 626. 

3. Users
	1. Looking up "akilahj" umd gives us https://drupaluser.umd.edu/users/akilahj, which tells us the full name of this directory id is Akilah Jackson. 