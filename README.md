# Incident Response Case File: Analysis of web breach on gacrux (172.31.4.249)

## Executive Summary
During a Digital Forensics and Incident Response (DFIR) investigation of the `botsv2` dataset in Splunk, an attack vector targeting the **MyBB** forum server (`gacrux` / `172.31.4.249`) was identified and fully reconstructed. 

The threat actor executed a multi-stage attack involving **SQL Injection (SQLi)**, **Cross-Site Request Forgery (CSRF)** targeting an authenticated administrator, and **unauthorized administrative access**. Subsequent post-exploitation analysis confirmed that while the attacker successfully created a backdoor administrative account and tampered with forum settings, no local web shell, SSH compromise, or host-level persistence was established on this endpoint.

---


