**3. What are potential problems/security impacts when updating firmware?**
	a. <mark style="background: #BBFABBA6;">Incorrectly/inconsistently verified signatures</mark>
	b. <mark style="background: #BBFABBA6;">Non-verified signatures</mark>
	c. <mark style="background: #BBFABBA6;">Leaking signature keys</mark>
	d. Mutual-authenticating update protocols
	e. <mark style="background: #BBFABBA6;">Allow firmware downgrade</mark>
	f. <mark style="background: #BBFABBA6;">TOCTOU attacks</mark> -> gap between checking the integrity of firmware and actually using it, allowing attackers to modify the firmware after it has been verified and before it is used.
	g. Signed packages


**4. What is a side channel attack? Give two examples with different side channels and explain them briefly**
	==**Un attacco side channel è un tipo di violazione della sicurezza che mira alle informazioni rilasciate involontariamente da un sistema durante la sua normale operatività**==. Invece di attaccare direttamente gli algoritmi crittografici o le vulnerabilità del software, ==**gli attacchi side channel sfruttano l'implementazione fisica di un sistema o le informazioni non intenzionali rilasciate dal suo comportamento**==.
	*Esempio:*
		- **Power Analysis Attack**, misuriamo il consumo di energia di un device mentre effettua operazioni di crittografia
		- **Timing Attack**, misuriamo il tempo impiegato dal sistema a fare determinate operazioni 


**6. You received a wireless home router from your ISP and suspect that it has hidden functionality in it that might be disadvantageous for you. You open up the router and discover a SoC controller chip and a memory chip in the device. Describe which steps you can take to analyze the firmware of the device in order to identify hidden functionality, back-doors and vulnerabilities.**
	1. extract the firmware from the memory chip
	2. reverse engineering of the firmware, in order to obtain the components, functionalitites, structure and the source code
	3. dynamic/static analysis of the firmware
	4. code review



**10. What are characteristics of the Android developer signatures?**
	a. <mark style="background: #BBFABBA6;">They can be handled by Google Play on behalf of developers</mark>
	b. They allow users to identify the developers
	c. They need to match the certificate on the developer's website
	d. <mark style="background: #BBFABBA6;">They are typically self signed</mark>



**15. What is true about secure boot?**
	a. If signature verification in a stage fails, boot continues with a warning message
	b. The bootloader allows booting of unauthorized operating systems
	c. <mark style="background: #BBFABBA6;">Allowed to set more protections, but not to remove protections on following stages</mark>
	d. The operating system verifies the bootloaders signature


**18. Which of these properties apply to the Android Runtime (ART)?**
	a. <mark style="background: #BBFABBA6;">The ART runs code that was compiled from Java or Kotlin source code.</mark>
	b. <mark style="background: #BBFABBA6;">The ART supports both ahead-of-time compilation and just-in-time compilation.</mark>
	c. <mark style="background: #BBFABBA6;">The ART is a register-based VM.</mark> (based on Dalvik VM)
	d. The ART requires developers to compile applications to native code (ELF) (no, we don't need to compile application to ELF)
