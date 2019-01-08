Usually, the simplest way is to let the persistence mechanism to generate the identity because the vast major of persistence mechanisms supports some kind of identity generation, like MySQL’s AUTO\_INCREMENT attribute or Oracle’s/Postgres sequences. This, although simple, have a major drawback: We  won’t have the identity of the entity until we persist it. So to some degree, if  we are going with persistence mechanism generated identities we will couple the identity operation with the underlying persistence store.



