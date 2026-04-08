# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static single-page birthday party invitation website for Anya's 6th birthday (Demon Hunter / K-Pop theme). The entire site is a single `index.html` file with inline CSS and JavaScript — no build tools, no frameworks, no dependencies.

## Architecture

- **index.html**: The complete application — markup, styles (`<style>` block), and scripts (`<script>` block)
- **Images**: `Amya_Huntrix.png` (character art at bottom of card), `anya_1.jpg` (photo in circular frame), `demon-hunter-2.png` (unused/available asset)
- **Fonts**: Google Fonts loaded via CDN (Cinzel Decorative, Nunito)
- **RSVP form**: Uses Netlify Forms (`data-netlify="true"`) with AJAX submission — deployed on Netlify
- **Starfield**: Canvas-based animated star background with twinkle effect

## Development

Open `index.html` directly in a browser — no server or build step required. For testing the RSVP form submission, deploy to Netlify (the form uses Netlify's built-in form handling).

## RSVP Confirmation Email Setup (EmailJS)

The site uses EmailJS to send confirmation emails from the browser when a guest RSVPs "yes". No backend or custom domain required — emails send from your Gmail.

### To finish setup:

1. Sign up at https://www.emailjs.com (free tier = 200 emails/month)
2. Go to **Email Services** > **Add New Service** > choose **Gmail** and connect your Gmail account
3. Note your **Service ID** (e.g. `service_abc123`)
4. Go to **Email Templates** > **Create New Template** and set up:
   - **To Email**: `{{to_email}}`
   - **Subject**: `You're coming to Anya's Birthday! 🎉⚔️`
   - **Body** (example):
     ```
     Hi {{guest_name}}!

     Thank you for RSVPing for {{child_name}}!

     📅 Saturday, May 23, 2026
     ⏰ 11:00 AM – 3:00 PM
     📍 12774 Hill Creek Rd, Montgomery, TX 77356

     Come dressed in your most powerful Demon Hunter look — purple, pink, gold & ALL the sparkle!

     — The Salian Family 💜
     ```
   - Note your **Template ID** (e.g. `template_xyz789`)
5. Go to **Account** > copy your **Public Key**
6. In `index.html`, replace the three placeholders:
   - `YOUR_PUBLIC_KEY` — your EmailJS public key
   - `YOUR_SERVICE_ID` — your EmailJS service ID
   - `YOUR_TEMPLATE_ID` — your EmailJS template ID

## TODO: Remaining Setup Steps

- [ ] Sign up at https://www.emailjs.com and connect Gmail (`thaoninh.email@gmail.com`)
- [ ] Create an email template with variables `{{to_email}}`, `{{guest_name}}`, `{{child_name}}`
- [ ] Replace `YOUR_PUBLIC_KEY` in `index.html` with your EmailJS public key
- [ ] Replace `YOUR_SERVICE_ID` in `index.html` with your EmailJS service ID
- [ ] Replace `YOUR_TEMPLATE_ID` in `index.html` with your EmailJS template ID
- [ ] Test the RSVP form on the deployed Netlify site (form submission won't work locally)
- [ ] Commit and push changes to trigger a Netlify redeploy

## Key Design Details

- Dark purple/gold color scheme (#0d0225 background, #f5c842 gold accents, #a259f7 purple, #f472b6 pink)
- Card is fixed 600px wide, centered on page
- Gradient text effects use `-webkit-background-clip: text`
- Honeypot spam protection on the RSVP form (`bot-field`)
