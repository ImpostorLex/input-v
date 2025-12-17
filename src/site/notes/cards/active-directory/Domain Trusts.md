---
{"dg-publish":true,"permalink":"/cards/active-directory/domain-trusts/"}
---

~ [[cards/active-directory/windows active directory\|Active Directory]]
## Domain Trusts
---
A forests is a collection of one or more domain trees inside an AD network domain. Domain Trusts are a mechanism for users in the network to gain access to other resources in the domain.

- Directional - The direction of the trust flows from a trusting domain to a trusted domain. Which way does trust flow?

	- **Trusting domain** = the domain that ALLOWS access (gives trust)
	- **Trusted domain** = the domain that RECEIVES access (is trusted)
	- Think: "Domain A trusts Domain B" means Domain A allows Domain B's users to access Domain A's resources

- Transitive - The trust relationship expands beyond just two domains to include other trusted domains. Does trust extend beyond two domains?

	- If A trusts B, and B trusts C, then A automatically trusts C
	- It "passes through" or "chains" the trust relationship

It is common to have a root or parent domain in a forest. For each regional office, sub or child domains are created such as `ZA.TRYHACKME.LOC`. 

**User in UK office needs access to THMSERVER1 in ZA domain:**

1. UK user authenticates to UK.TRYHACKME.LOC
2. UK domain has trust with ROOT (TRYHACKME.LOC)
3. ROOT has trust with ZA (ZA.TRYHACKME.LOC)
4. Through **transitive trust**, UK user can be granted permissions in ZA domain
5. ZA admin can add UK user to THMSERVER1's access control list
6. UK user can now access THMSERVER1

**Without transitive trust:** You'd need a direct trust between ZA and UK domains. With it, the ROOT domain acts as a "bridge."

The trust between parent and child domain is bidirectional. This is an intended behaviour, as an attacker, you can exploit this trusts to compromise parent domain if we have compromised a child domain.