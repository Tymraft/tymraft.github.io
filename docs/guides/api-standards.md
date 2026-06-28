---
title: API Standards Guide
type: Guide
status: Draft
author: Ethan Hollins
date: 2026-06-28
---

# API Standards Guide

This guide outlines the API design standards for Tymraft projects.

## Conventions

- Use RESTful resource naming
- Return consistent JSON error shapes
- Version all endpoints under `/api/v1/`

## Authentication

All endpoints require a Bearer token in the `Authorization` header.
