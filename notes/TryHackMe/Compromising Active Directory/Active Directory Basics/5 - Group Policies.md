**Group Policy Objects (GPO)** are simply a collection of settings that can be applied to OUs. 
GPOs can contain policies aimed at either users or computers, allowing you to set a baseline on specific machines and identities.

To configure GPOs, you can use the **Group Policy Management** tool, available from the start menu:
- ![[Pasted image 20250128141548.png]]


The first thing you will see when opening it is your complete OU hierarchy, as defined before. 

To configure Group Policies, you first create a GPO under **Group Policy Objects** and then link it to the OU where you want the policies to apply. 

As an example, you can see there are some already existing GPOs in your machine:
- ![[Pasted image 20250128141631.png]]


The `Default Domain Policy` and `RDP Policy` are linked to the `thm.local` domain as a whole, and the `Default Domain Controllers Policy` is linked to the `Domain Controllers` OU only. 

Something important to have in mind is that any GPO will apply to the linked OU and any sub-OUs under it. 
- For example, the `Sales` OU will still be affected by the `Default Domain Policy`.

Let's examine the `Default Domain Policy` to see what's inside a GPO. 

The first tab you'll see when selecting a GPO shows its **scope**, which is where the GPO is linked in the AD. For the current policy, we can see that it has only been linked to the `thm.local` domain:
- ![[Pasted image 20250128141807.png]]


As you can see, you can also apply **Security Filtering** to GPOs so that they are only applied to specific users/computers under an OU. 
- By default, they will apply to the **Authenticated Users** group, which includes all users/PCs.

The **Settings** tab includes the actual contents of the GPO and lets us know what specific configurations it applies.
In this case, the `Default Domain Policy` only contains Computer Configurations:
- ![[Pasted image 20250128141901.png]]


Since this GPO applies to the whole domain, any change to it would affect all computers. 
Let's change the minimum password length policy to require users to have at least 10 characters in their passwords. 

To do this, right-click the GPO and select **Edit**:
- ![[Pasted image 20250128142114.png]]


This will open a new window where we can navigate and edit all the available configurations. 

To change the minimum password length, go to `Computer Configurations -> Policies -> Windows Setting -> Security Settings -> Account Policies -> Password Policy` and change the required policy value:
- ![[Pasted image 20250128142314.png]]


If more information on any of the policies is needed, you can double-click them and read the **Explain** tab on each of them:
- ![[Pasted image 20250128142354.png]]

# GPO Distribution
GPOs are distributed to the network via a network share called `SYSVOL`, which is stored in the DC. 

The SYSVOL share points by default to the `C:\Windows\SYSVOL\sysvol\` directory on each of the DCs in our network.

## enforce changes on computers
Once a change has been made to any GPOs, it might take up to 2 hours for computers to catch up.

If you want to force any particular computer to sync its GPOs immediately, you can always run the following command on the desired computer:
```powershell
gpupdate /force
```


# Creating some GPOs for THM Inc.

You have been tasked with implementing some GPOs to allow you to:
1. **Block non-IT users from accessing the Control Panel**.
2. **Make workstations and servers lock their screen automatically after 5 minutes of user inactivity** 

Let's focus on each of those and define what policies we should enable in each GPO and where they should be linked.

**_Restrict Access to Control Panel_**
We want to restrict access to the Control Panel across all machines to only the users that are part of the IT department. 

Let's create a new GPO called `Restrict Control Panel Access` and open it for editing. 
- ![[Pasted image 20250128143116.png]]

Since we want this GPO to apply to specific users, we will look under `User Configuration` for the following policy:
- ![[Pasted image 20250128142739.png]]

So we enabled it but we need to link it to all users except **IT users**. In this case, we will link the `Marketing`, `Management` and `Sales`. Just go on `Group Policy Objects` and drag it into the OUs
- ![[Pasted image 20250128143627.png]]

**_Auto Lock Screen GPO_**
For the first GPO, regarding screen locking for workstations and servers, we could directly apply it over the `Workstations`, `Servers` and `Domain Controllers` OUs we created previously.

While this solution should work, an alternative consists of simply applying the GPO to the root domain, as we want the GPO to affect all of our computers. 

Since the `Workstations`, `Servers` and `Domain Controllers` OUs are all child OUs of the root domain, they will inherit its policies.

**Note:** You might notice that if our GPO is applied to the root domain, it will also be inherited by other OUs like `Sales` or `Marketing`. Since these OUs contain users only, any Computer Configuration in our GPO will be ignored by them.

Let's create a new GPO, call it `Auto Lock Screen`, and edit it. The policy to achieve what we want is located in the following route:
- ![[Pasted image 20250128143802.png]]

After closing the GPO editor, we will link the GPO to the root domain by dragging the GPO to it as before:
- ![[Pasted image 20250128143837.png]]

> **Note:** If you created and linked the GPOs, but for some reason, they still don't work, remember you can run `gpupdate /force` to force GPOs to be updated.


