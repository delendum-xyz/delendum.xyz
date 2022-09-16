---
layout: plain
title: "Delendum - Incubation"
description: "We support inventions in blockchain infrastructure, private computing, and zero-knowledge proof
applications"
---

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.1/dist/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.1/dist/js/bootstrap.bundle.min.js"></script>
<div class="main-content-incubator">
    <h2>Delendum ZKP Incubator</h2>
    <p class="text-white text-research-para">
        Delendum focuses on zero-knowledge proof innovation. We are launching an incubator that will help
        early-stage
        founders build their technology with the advice and support of our partners. If you are interested in
        applying
        to the incubator, please fill out the form below. We will consider applications on a rolling basis. In the
        meantime, you can join our discussion group chat or learn more about us here.
    </p>
    <form action="" class="needs-validation" novalidate>
        <div class="form-group">
            <div class="form-label">
                <label for="full_name">Full Name *</label>
            </div>
            <div class="form-input ">
                <input type="text" class="form-control" id="full_name" placeholder="Your Full Name" required>
                <div class="invalid-feedback">Please fill out this field.</div>
            </div>
        </div>
        <div class="form-group">
            <div class="form-label">
                <label for="email">Email *</label>
            </div>
            <div class="form-input">
                <input type="email" class="form-control" id="email" placeholder="Your Email" required>
                <div class="invalid-feedback">Please fill out this field.</div>
            </div>
        </div>
        <div class="form-group">
            <div class="form-label">
                <label for="personal_git">Personal Github Profile</label>
            </div>
            <div class="form-input">
                <input type="text" class="form-control" placeholder="Your answer" id="personal_git">
            </div>
        </div>
        <div class="form-group">
            <div class="form-label">
                <label for="edu_back">Educational Background *</label>
            </div>
            <div class="form-input">
                <input type="text" class="form-control" id="edu_back" placeholder="Your answer" required>
                <div class="invalid-feedback">Please fill out this field.</div>
            </div>
        </div>
        <div class="form-group">
            <div class="form-label">
                <label for="proj_name">Project Name *</label>
            </div>
            <div class="form-input">
                <input type="text" class="form-control" id="proj_name" placeholder="Your answer" required>
                <div class="invalid-feedback">Please fill out this field.</div>
            </div>
        </div>
        <div class="form-group">
            <div class="form-label">
                <label for="proj_des">Project Description *</label>
            </div>
            <div class="form-input">
                <input type="text" class="form-control" id="proj_des" placeholder="Your answer" required>
                <div class="invalid-feedback">Please fill out this field.</div>
            </div>
        </div>
        <div class="form-group">
            <label for="proj_stage">Project Stage</label>
            <div class="form-radio">
                <input type="radio" class="form-check-input" id="idea" name="proj_stage" value="Idea">
                <label for="idea" class="form-check-label">Idea</label><br>
                <input type="radio" class="form-check-input" id="tech_paper" name="proj_stage"
                    value="Technical Whitepaper">
                <label for="tech_paper" class="form-check-label">Technical Whitepaper</label><br>
                <input type="radio" class="form-check-input" id="mvp_concept" name="proj_stage"
                    value="MVP-Proof of Concept">
                <label for="mvp_concept" class="form-check-label">MVP-Proof of Concept</label><br>
                <input type="radio" class="form-check-input" id="pri_launch" name="proj_stage" value="Private launch">
                <label for="pri_launch" class="form-check-label">Private launch</label><br>
                <input type="radio" class="form-check-input" id="pub_launch" name="proj_stage" value="Public launch">
                <label for="pub_launch" class="form-check-label">Public launch</label><br>
            </div>
        </div>
        <div class="form-group">
            <div class="form-label">
                <label for="add-material">Additional Materials</label>
            </div>
            <div class="form-input custom-button">
                <input type="file" class="form-control" id="add_material">
            </div>
        </div>
        <div class="form-group">
            <div class="form-label">
                <label for="proj_des">Link to project Github (if open-sourced)</label>
            </div>
            <div class="form-input">
                <input type="text" class="form-control" id="proj_des" placeholder="Your answer">
            </div>
        </div>
        <hr />
        <div class="form-group">
            <button type="submit" class="btn btn-primary">Submit</button>
        </div>
    </form>
</div>

<script>
    (function () {
        'use strict'
        var forms = document.querySelectorAll('.needs-validation')

        // Loop over them and prevent submission
        Array.prototype.slice.call(forms)
            .forEach(function (form) {
                form.addEventListener('submit', function (event) {
                    if (!form.checkValidity()) {
                        event.preventDefault()
                        event.stopPropagation()
                    }

                    form.classList.add('was-validated')
                }, false)
            })
    })()
</script>